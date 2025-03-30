<template>
  <div fluid>
    <v-row v-show="!dialog">
      <v-col md="8" cols="12" class="pb-2 pr-0">
        <v-card
          class="main mx-auto grey lighten-5 mt-3 p-3 pb-16 overflow-y-auto"
          style="max-height: 94vh; height: 94vh"
        >
          <div>
            <v-row>
              <v-col md="7" cols="12">
                <h3 style="margin-top: 10px">
                  <strong>{{ __("Sales Order List") }}</strong>
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
                    :label="__('Search by Part of Order Name, Amount or Table Name')"
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
                <v-btn block color="warning" @click="get_list_of_orders" dark>
                  {{ __("Search") }}
                </v-btn>
              </v-col>
            </v-row>
            <v-divider></v-divider>

            <!-- جدول عرض الطلبات -->
            <v-data-table
              :headers="order_headers"
              :items="orders_data"
              item-key="name"
              class="elevation-1 mt-0"
              :loading="orders_loading"
              checkbox-color="primary"
            >
              <!-- slot مخصص لعرض كل صف -->
              <template v-slot:item="{ item }">
                <tr
                  :class="{ 'selected-row': isSelected(item) }"
                  @click="selectOrder(item)"
                >
                  <td
                    v-for="header in order_headers"
                    :key="header.value"
                  >
                    <!-- فحص قيمة الحقل؛ إذا كانت status نستخدم v-chip مع اللون -->
                    <template v-if="header.value === 'status'">
                      <v-chip variant="elevated" :color="item.color">
                        {{ item.status }}
                      </v-chip>
                    </template>
                    <!-- وإلا نعرض قيمة الحقل مباشرة -->
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
      <!-- لوحة جانبية لعرض تفاصيل الطلب -->
      <v-col md="4" cols="12" class="pb-2 pr-0" v-if="selected_orders.length">
        <v-card
          class="order-details mx-auto grey lighten-4 mt-3 pa-4 elevation-3"
          style="max-height: 94vh; height: 94vh; overflow-y: auto"
        >
          <!-- Header -->
          <v-row class="mb-3" align="center">
            <v-col cols="12">
              <h3 class="headline primary--text mb-1">
                <v-icon left color="primary">mdi-file-document-outline</v-icon>
                {{ __("Order Details") }}
              </h3>
              <v-divider></v-divider>
            </v-col>
          </v-row>

          <!-- بيانات الطلب الأساسية -->
          <v-container fluid>
            <v-row class="mb-2">
              <v-col cols="6">
                <span class="grey--text text--darken-2 font-weight-bold">{{ __("Order Name:") }}</span>
              </v-col>
              <v-col cols="6" class="text-right">
                <span>{{ selected_orders[0].name }}</span>
              </v-col>
            </v-row>

            <v-row class="mb-2">
              <v-col cols="6">
                <span class="grey--text text--darken-2 font-weight-bold">{{ __("Customer:") }}</span>
              </v-col>
              <v-col cols="6" class="text-right">
                <span>{{ selected_orders[0].customer }}</span>
              </v-col>
            </v-row>

            <v-row class="mb-2">
              <v-col cols="6">
                <span class="grey--text text--darken-2 font-weight-bold">{{ __("Status:") }}</span>
              </v-col>
              <v-col cols="6" class="text-right">
                <v-chip :color="selected_orders[0].color" dark>{{ selected_orders[0].status }}</v-chip>
              </v-col>
            </v-row>

            <v-row class="mb-2">
              <v-col cols="6">
                <span class="grey--text text--darken-2 font-weight-bold">{{ __("Order Date:") }}</span>
              </v-col>
              <v-col cols="6" class="text-right">
                <span>{{ selected_orders[0].transaction_date }}</span>
              </v-col>
            </v-row>

            <v-row class="mb-2">
              <v-col cols="6">
                <span class="grey--text text--darken-2 font-weight-bold">{{ __("Due Date:") }}</span>
              </v-col>
              <v-col cols="6" class="text-right">
                <span>{{ selected_orders[0].delivery_date }}</span>
              </v-col>
            </v-row>

            <v-divider class="my-3"></v-divider>

            <!-- المجموع الكلي -->
            <v-row class="mb-3" align="center" justify="center">
              <v-col cols="12" class="text-center">
                <h4 class="primary--text font-weight-bold">
                  <v-icon left color="primary">mdi-currency-usd</v-icon>
                  {{ currencySymbol(pos_profile_details.currency) }}{{ selected_orders[0].grand_total.toLocaleString() }}
                </h4>
              </v-col>
            </v-row>

            <v-divider class="my-3"></v-divider>

            <!-- عرض عناصر الطلب -->
            <v-row class="mb-2">
              <v-col cols="12">
                <h4 class="primary--text mb-2">
                  <v-icon left color="primary">mdi-cart-outline</v-icon>
                  {{ __("Order Items") }}
                </h4>
              </v-col>
            </v-row>
            <v-data-table
              dense
              :headers="order_items_headers"
              :items="selected_order_items"
              item-key="name"
              class="elevation-0"
              hide-default-footer
              disable-pagination
            ></v-data-table>
          </v-container>

          <v-divider class="my-3"></v-divider>

          <!-- زر الطباعة -->
          <v-btn block color="primary" dark @click="print_invoice">
            <v-icon left>mdi-printer</v-icon>
            {{ __("Print Order") }}
          </v-btn>
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
  data() {
    return {
      dialog: false,
      pos_profile: "",
      pos_profile_details: {},

      orders_data: [],
      includeDrafts: false,
      search: "",
      orders_loading: false,
      selected_orders: [],
      order_loading: false,

      order_headers: [
        {
          text: __("Order Name"),
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
          value: "transaction_date",
        },
        {
          text: __("Due Date"),
          align: "start",
          sortable: true,
          value: "delivery_date",
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
      ],

      selected_order_items: [],
      order_items_headers: [
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
    // عند اختيار صف معين
    selectOrder(order) {
      this.selected_orders = [order];
      this.get_order_items();
    },
    // التحقق هل الصف الحالي هو المحدد
    isSelected(item) {
      return this.selected_orders.length && this.selected_orders[0].name === item.name;
    },

    // جلب قائمة الطلبات
    get_list_of_orders() {
      this.orders_loading = true;
      return frappe.call({
        method: "posawesome.posawesome.api.order.get_orders_list",
        args: { term: this.search.trim(), include_drafts: this.includeDrafts },
        callback: (r) => {
          if (r.message) {
            this.orders_data = r.message.map((el) => {
              // تحديد لون الحالة
              el.color = ["Cancelled", "Closed"].includes(el.status)
                ? "red"
                : el.status === "Completed"
                ? "green"
                : "yellow";
              return el;
            });
          } else {
            console.log("error");
          }
          this.orders_loading = false;
        },
      });
    },

    // جلب تفاصيل الطلب (العناصر)
    get_order_items() {
      if (!this.selected_orders.length) return;
      return frappe.call({
        method: "posawesome.posawesome.api.order.get_order_items",
        args: { order: this.selected_orders[0] },
        callback: (r) => {
          if (r.message) {
            this.selected_order_items = r.message.map((el) => {
              el.rate = `${el.rate} ${this.pos_profile_details.currency}`;
              el.amount = `${el.amount} ${this.pos_profile_details.currency}`;
              return el;
            });
          }
        },
      });
    },

    // التحقق من وجود جلسة عمل مفتوحة
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

    // طباعة الفاتورة
    print_invoice() {
      this.load_print_page(this.selected_orders[0].name);
    },
    load_print_page(order_name) {
      const print_format =
        this.pos_profile.print_format_for_online || this.pos_profile.print_format;
      const letter_head = this.pos_profile.letter_head || 0;
      const url =
        frappe.urllib.get_base_url() +
        "/printview?doctype=Sales%20Order&name=" +
        order_name +
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
  },

  watch: {
    selected_orders(value) {
      if (value.length) {
        this.get_order_items();
      } else {
        this.selected_order_items = [];
      }
    },
    includeDrafts() {
      this.get_list_of_orders();
    },
  },

  mounted() {
    this.$nextTick(() => {
      this.check_opening_entry();
      this.get_list_of_orders();
    });
  },
};
</script>

<style>
/* لون الصف المحدد */
.selected-row {
  background-color: rgba(25, 118, 210, 0.2) !important; /* لون أزرق فاتح */
}

/* أي تنسيقات إضافية لمحاذاة النص */
input[total_of_diff],
input[payments_methods],
input[total_selected_payments],
input[total_selected_invoices],
input[total_selected_mpesa_payments] {
  text-align: right;
}
</style>
