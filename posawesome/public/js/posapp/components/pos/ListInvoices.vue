<template>
  <v-row justify="center">
    <v-dialog v-model="invoicesListDialog" max-width="800px">
      <v-card>
        <v-card-title>
          <span class="headline primary--text">{{ __("Invoices List") }}</span>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container>
            <v-row>
              <v-col cols="12" class="pa-1">
                <div class="mx-2 my-5">
                  <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    :label="__('Search by Part of Invoice Name, Amount or Table Name')"
                    single-line
                    hide-details
                  ></v-text-field>
                </div>
                <template>
                  <v-container>
                    <v-checkbox
                      v-model="includeDrafts"
                      :label="__('Include Drafts')"
                    ></v-checkbox>
                  </v-container>
                  <v-data-table
                    :headers="headers"
                    :items="invoices_data"
                    item-key="name"
                    class="elevation-1"
                    :single-select="singleSelect"
                    :footer-props="{ 'items-per-page-options': [5, -1] }"
                    show-select
                    v-model="selected"
                  >
                  </v-data-table>
                </template>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="error" dark @click="close_dialog">{{ __("Close") }}</v-btn>
          <v-btn color="success" dark @click="print_invoice">{{ __("Print") }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
export default {
  data: () => ({
    invoicesListDialog: false,
    singleSelect: true,
    includeDrafts: false,
    selected: [],
    invoices_data: [],
    search: "",
    pos_profile: null, // تمت إضافته هنا
    headers: [
      {
        text: __("Customer"),
        value: "customer",
        align: "start",
        filterable: true,
        sortable: true,
      },
      {
        text: __("Date"),
        align: "start",
        sortable: true,
        value: "posting_date",
      },
      {
        text: __("Status"),
        align: "start",
        sortable: true,
        value: "status",
      },
      {
        text: __("Invoice"),
        value: "name",
        align: "start",
        sortable: true,
      },
      {
        text: __("Amount"),
        value: "grand_total",
        align: "start",
        sortable: false,
      },
    ],
  }),

  watch: {
    search(value) {
      this.search_invoice(value);
    },
    includeDrafts(value) {
      this.search_invoice(value);
    },
  },

  methods: {
    close_dialog() {
      this.invoicesListDialog = false;
    },

    print_invoice() {
      if (this.selected.length > 0) {
        this.load_print_page(this.selected[0].name);
      }
    },

    load_print_page(invoice_name) {
      const print_format =
        this.pos_profile.print_format_for_online || this.pos_profile.print_format;
      const letter_head = this.pos_profile.letter_head || 0;
      const url = `${frappe.urllib.get_base_url()}/printview?doctype=Sales%20Invoice&name=${
        invoice_name
      }&trigger_print=1&format=${print_format}&no_letterhead=${letter_head}`;
      
      const printWindow = window.open(url, "Print");
      printWindow.addEventListener(
        "load",
        () => {
          printWindow.print();
        },
        true
      );
    },

    search_invoice() {
      frappe.call({
        method: "posawesome.posawesome.api.invoice.get_invoices_list",
        args: { 
          term: this.search.trim(), 
          include_drafts: this.includeDrafts 
        },
        callback: (r) => {
          this.invoices_data = r.message.map((el) => ({
            ...el,
            status: `(${el.docstatus === 0 ? "Draft" : "Submitted"})`
          }));
        },
      });
    },
  },

  created() {
    // تصحيح اسم eventBus وإضافة الـ listeners
    this.eventBus.on("register_pos_profile", (data) => {
      this.pos_profile = data.pos_profile;

      if (this.pos_profile?.posa_enable_pos_restaurant_table) {
        this.headers.push({
          text: __("Table"),
          value: "posa_pos_restaurant_table",
          align: "start",
          sortable: true,
        });
      }
    });

    this.eventBus.on("open_invoices_list", () => {
      this.invoicesListDialog = true;
      this.selected = [];
      this.search_invoice();
    });

    this.search_invoice();
  },

  beforeUnmount() {
    // تنظيف الـ event listeners
    this.eventBus.off("register_pos_profile");
    this.eventBus.off("open_invoices_list");
  },
};
</script>