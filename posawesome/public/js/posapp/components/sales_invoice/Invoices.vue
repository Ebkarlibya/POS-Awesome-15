<template>
  <div fluid>
    <v-row v-show="!dialog">
      <!-- قائمة الفواتير -->
      <v-col md="8" cols="12" class="pb-2 pr-0">
        <v-card
          class="main mx-auto grey lighten-5 mt-3 p-3 pb-16 overflow-y-auto"
          style="max-height: 94vh; height: 94vh"
        >
          <div>
            <v-row>
              <v-col md="7" cols="12">
                <h3 style="margin-top: 10px">
                  <strong>{{ __("Sales Invoice List") }}</strong>
                </h3>
                <v-divider></v-divider>
              </v-col>
            </v-row>
            <v-row>
              <v-col md="6" cols="12">
                <div class="mx-2 my-5">
                  <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    :label="__('Search by Part of Invoice Name, Amount or Table Name')"
                    single-line
                    hide-details
                  ></v-text-field>
                </div>
              </v-col>
              <v-col md="2" cols="12">
                <v-checkbox
                  v-model="includeDrafts"
                  label="Include Drafts"
                  color="success"
                  :value="!includeDrafts"
                  hide-details
                ></v-checkbox>
              </v-col>
              <v-col md="4" cols="12">
                <v-btn block color="warning" @click="get_list_of_invoices" dark>
                  {{ __("Search") }}
                </v-btn>
              </v-col>
            </v-row>
            <v-divider></v-divider>
            <!-- جدول عرض الفواتير مع تخصيص الصفوف -->
            <v-data-table
              :headers="invoice_headers"
              :items="invoice_data"
              item-key="name"
              class="elevation-1 mt-0"
              :loading="invoice_loading"
              checkbox-color="primary"
            >
              <template v-slot:item="{ item }">
                <tr
                  :class="{ 'selected-row': isSelected(item) }"
                  @click="selectInvoice(item)"
                >
                  <td v-for="header in invoice_headers" :key="header.value">
                    <!-- تخصيص عرض بعض الحقول -->
                    <template v-if="header.value === 'grand_total'">
                      {{ currencySymbol(item.currency) }} {{ formatCurrency(item.grand_total) }}
                    </template>
                    <template v-else-if="header.value === 'outstanding_amount'">
                      <span class="primary--text">
                        {{ currencySymbol(item.currency) }} {{ formatCurrency(item.outstanding_amount) }}
                      </span>
                    </template>
                    <template v-else-if="header.value === 'status'">
                      <v-chip variant="elevated" :color="item.color">
                        {{ item.status }}
                      </v-chip>
                    </template>
                    <template v-else>
                      {{ item[header.value] }}
                    </template>
                  </td>
                </tr>
              </template>
            </v-data-table>
            <v-divider></v-divider>
          </div>
        </v-card>
      </v-col>

      <!-- لوحة جانبية لعرض تفاصيل الفاتورة -->
      <v-col md="4" cols="12" class="pb-2 pr-0" v-if="selected_invoices.length">
        <v-card
          class="invoice-details mx-auto grey lighten-4 mt-3 pa-4 elevation-3"
          style="max-height: 94vh; height: 94vh; overflow-y: auto"
        >
          <!-- Header -->
          <v-row align="center" class="mb-3">
            <v-col cols="12">
              <h3 class="headline primary--text mb-1">
                <v-icon left color="primary">mdi-file-document-outline</v-icon>
                {{ __("Sales Invoice Details") }}
              </h3>
              <v-divider></v-divider>
            </v-col>
          </v-row>

          <!-- Invoice Basic Info -->
          <v-container fluid>
            <v-row class="mb-2">
              <v-col cols="6">
                <span class="grey--text text--darken-2 font-weight-bold">
                  {{ __("Invoice Name:") }}
                </span>
              </v-col>
              <v-col cols="6" class="text-right">
                <span>{{ selected_invoices[0].name }}</span>
              </v-col>
            </v-row>
            <v-row class="mb-2" v-if="selected_invoices[0].customer">
              <v-col cols="6">
                <span class="grey--text text--darken-2 font-weight-bold">
                  {{ __("Customer:") }}
                </span>
              </v-col>
              <v-col cols="6" class="text-right">
                <span>{{ selected_invoices[0].customer }}</span>
              </v-col>
            </v-row>

            <v-divider class="my-3"></v-divider>

            <!-- Totals Section -->
            <v-row class="mb-3" align="center" justify="center">
              <v-col cols="12" class="text-center">
                <h4 class="primary--text font-weight-bold">
                  <v-icon left color="primary">mdi-currency-usd</v-icon>
                  {{ currencySymbol(pos_profile_details.currency) }}{{ selected_invoices[0].grand_total.toLocaleString() }}
                </h4>
              </v-col>
            </v-row>
            <v-row class="mb-3">
              <v-col cols="6">
                <span class="grey--text font-weight-bold">{{ __("Outstanding Amount:") }}</span>
              </v-col>
              <v-col cols="6" class="text-right">
                <span>{{ currencySymbol(pos_profile_details.currency) }}{{ formatCurrency(selected_invoices[0].outstanding_amount) }}</span>
              </v-col>
            </v-row>

            <v-divider class="my-3"></v-divider>

            <!-- Invoice Items Section -->
            <v-row class="mb-2">
              <v-col cols="12">
                <h4 class="primary--text mb-2">
                  <v-icon left color="primary">mdi-cart-outline</v-icon>
                  {{ __("Invoice Items") }}
                </h4>
              </v-col>
            </v-row>
            <v-data-table
              dense
              :headers="invoice_items_headers"
              :items="selected_invoice_items"
              item-key="name"
              class="elevation-0"
              hide-default-footer
              disable-pagination
            ></v-data-table>
          </v-container>

          <v-divider class="my-3"></v-divider>

          <!-- Action Buttons -->
          <v-row>
            <v-col cols="12">
              <v-btn block color="primary" dark @click="print_invoice">
                <v-icon left>mdi-printer</v-icon>
                {{ __("Print Invoice") }}
              </v-btn>
              <v-btn
                v-if="pos_profile_details.posa_enable_warranty_print_system && selected_invoices[0] && selected_invoices[0].posa_has_warranty === 'Yes'"
                block
                color="orange"
                dark
                class="mt-3"
                @click="print_warranty_invoice"
              >
                <v-icon left>mdi-printer</v-icon>
                {{ __("Print Warranty") }}
              </v-btn>
            </v-col>
          </v-row>
        </v-card>
</v-col>
    </v-row>
  </div>
</template>

<script>
import format from "../../format";
import Customer from "../pos/Customer.vue";
import UpdateCustomer from "../pos/UpdateCustomer.vue";
export default {
  mixins: [format],
  components: { Customer, UpdateCustomer },
  data: function () {
    return {
      dialog: false,
      pos_profile: "",
      pos_profile_details: {},
      invoice_data: [],
      includeDrafts: false,
      search: "",
      selected_invoices: [],
      invoice_loading: false,
      invoice_headers: [
        {
          text: __("Invoice Name"),
          align: "start",
          sortable: true,
          value: "name",
        },
        {
          text: __("Customer"),
          align: "start",
          sortable: true,
          value: "customer",
        },
        {
          text: __("Date"),
          align: "start",
          sortable: true,
          value: "posting_date",
        },
        {
          text: __("Due Date"),
          align: "start",
          sortable: true,
          value: "due_date",
        },
        {
          text: __("Total"),
          align: "end",
          sortable: true,
          value: "grand_total",
        },
        {
          text: __("Status"),
          align: "end",
          sortable: true,
          value: "status",
        },
        {
          text: __("Outstanding"),
          align: "end",
          sortable: true,
          value: "outstanding_amount",
        },
        {
          text: __("Has Warranty"),
          align: "end",
          sortable: true,
          value: "posa_has_warranty",
        },
      ],
      selected_invoice_items: [],
      invoice_items_headers: [
        {
          text: __("Item Name"),
          align: "start",
          sortable: true,
          value: "item_code",
        },
        {
          text: __("Qty"),
          align: "start",
          sortable: true,
          value: "qty",
        },
        {
          text: __("Rate"),
          align: "end",
          sortable: true,
          value: "rate",
        },
        {
          text: __("Amount"),
          align: "end",
          sortable: true,
          value: "amount",
        },
      ],
    };
  },
  methods: {
    // دالة لتحديد فاتورة واحدة عند النقر عليها
    selectInvoice(invoice) {
      this.selected_invoices = [invoice];
      this.get_invoice_items();
    },
    // دالة للتحقق مما إذا كان الفاتورة المحددة هي نفس العنصر الحالي
    isSelected(item) {
      return this.selected_invoices.length && this.selected_invoices[0].name === item.name;
    },
    get_list_of_invoices() {
      this.invoice_loading = true;
      return frappe.call({
        method: "posawesome.posawesome.api.invoice.get_invoices_list",
        args: { term: this.search.trim(), include_drafts: this.includeDrafts },
        callback: (r) => {
          if (r.message) {
            this.invoice_data = r.message.map((el) => {
              el.color =
                el.status == "Unpaid" || el.status === "Overdue"
                  ? "red"
                  : el.status === "Paid"
                  ? "green"
                  : "yellow";
              return el;
            });
          }
          this.invoice_loading = false;
        },
      });
    },
    get_invoice_items() {
      return frappe.call({
        method: "posawesome.posawesome.api.invoice.get_invoice_items",
        args: { invoice: this.selected_invoices[0] },
        callback: (r) => {
          if (r.message) {
            this.selected_invoice_items = r.message.map((el) => {
              el.rate = `${el.rate} ${this.pos_profile_details.currency}`;
              el.amount = `${el.amount} ${this.pos_profile_details.currency}`;
              return el;
            });
          }
        },
      });
    },
    check_opening_entry() {
      return frappe
        .call("posawesome.posawesome.api.posapp.check_opening_shift", {
          user: frappe.session.user,
        })
        .then((r) => {
          if (r.message) {
            this.pos_profile_details = r.message.pos_profile;
            this.pos_profile = r.message.pos_profile.name;
          }
        });
    },
    print_invoice() {
      this.load_print_page(this.selected_invoices[0].name);
    },
    load_print_page(invoice_name) {
      const print_format =
        this.pos_profile_details.print_format_for_online || this.pos_profile_details.print_format;
      const letter_head = this.pos_profile_details.letter_head || 0;
      const url =
        frappe.urllib.get_base_url() +
        "/printview?doctype=Sales%20Invoice&name=" +
        invoice_name +
        "&trigger_print=1" +
        "&format=" +
        print_format +
        "&no_letterhead=" +
        letter_head;
      const printWindow = window.open(url, "Print");
      printWindow.addEventListener(
        "load",
        function () {
          printWindow.print();
        },
        true
      );
    },
    print_warranty_invoice() {
      if (
        this.selected_invoices.length &&
        this.selected_invoices[0].posa_has_warranty === "Yes"
      ) {
        this.load_warranty_print_page(this.selected_invoices[0].name);
      }
    },
    load_warranty_print_page(invoice_name) {
      const letter_head = this.pos_profile_details.letter_head || 0;
      const url =
        frappe.urllib.get_base_url() +
        "/printview?doctype=Sales%20Invoice&name=" +
        invoice_name +
        "&trigger_print=1" +
        "&format=" +
        this.pos_profile_details.posa_warranty_print_format +
        "&no_letterhead=" +
        letter_head;
      const printWindow = window.open(url, "PrintWarranty");
      printWindow.addEventListener(
        "load",
        function () {
          printWindow.print();
        },
        true
      );
    },
  },
  watch: {
    selected_invoices(value) {
      if (value.length) {
        this.get_invoice_items();
      } else {
        this.selected_invoice_items = [];
      }
    },
    includeDrafts() {
      this.get_list_of_invoices();
    },
  },
  mounted() {
    this.$nextTick(() => {
      this.check_opening_entry();
      this.get_list_of_invoices();
    });
  },
};
</script>

<style>
/* تلوين الصف المحدد */
.selected-row {
  background-color: rgba(25, 118, 210, 0.2) !important;
}

/* تنسيقات إضافية لمحاذاة النص داخل بعض الحقول */
input[total_of_diff],
input[payments_methods],
input[total_selected_payments],
input[total_selected_invoices],
input[total_selected_mpesa_payments] {
  text-align: right;
}
</style>
