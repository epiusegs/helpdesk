<template>
  <div class="flex flex-1 flex-col overflow-hidden overflow-y-auto border-b">
    <div
      v-for="o in options"
      :key="o.field"
      class="flex items-center gap-2 px-6 pb-1 leading-5 first:mt-3"
    >
      <Tooltip :text="o.label">
        <div class="w-[106px] shrink-0 truncate text-sm text-gray-600">
          {{ o.label }}
        </div>
      </Tooltip>
      
      <!-- Use a Rich Text Editor for the description field -->
      <QuillEditor
        v-if="o.field === 'description'"
        v-model="ticket[o.field]"
        theme="snow"
        class="form-control"
        @change="update(o.field, $event)"
      />
      
      <!-- Other fields remain unchanged -->
      <FormControl
        v-else-if="o.type === 'textarea'"
        class="form-control"
        :type="o.type"
        :value="ticket[o.field]"
        variant="subtle"
        rows="2"
        @change="update(o.field, $event.target.value, $event)"
      />
      <FormControl
        v-else-if="o.type === 'select'"
        class="form-control"
        :type="o.type"
        :value="ticket[o.field]"
        :options="customers?.data"
        @change="update(o.field, $event.target.value)"
      />
      <Autocomplete
        v-else
        class="form-control"
        :options="o.store.dropdown"
        :placeholder="`Add ${o.label}`"
        :value="ticket[o.field]"
        @change="update(o.field, $event.value)"
      />
    </div>
    <UniInput2
      v-for="field in customFields"
      :key="field.fieldname"
      :field="field"
      :value="ticket[field.fieldname]"
      @change="(data) => update(data.fieldname, data.value)"
    />
  </div>
</template>

<script setup lang="ts">
import { computed } from "vue";
import { createResource, FormControl, Tooltip } from "frappe-ui";
import { Autocomplete, QuillEditor } from "frappe-ui"; // Import QuillEditor component
import { useTeamStore } from "@/stores/team";
import { useTicketPriorityStore } from "@/stores/ticketPriority";
import { useTicketTypeStore } from "@/stores/ticketType";
import UniInput2 from "@/components/UniInput2.vue";
import { createToast } from "@/utils";
import { Field, FieldValue } from "@/types";

const emit = defineEmits(["update"]);

const props = defineProps({
  ticket: {
    type: Object,
    required: true,
  },
});

const customers = createResource({
  url: "helpdesk.utils.get_customer",
  params: {
    contact: props.ticket.raised_by,
  },
  auto: true,
  transform: (data: Array<object>) => {
    return data.map((d: object) => ({
      label: d,
      value: d,
    }));
  },
});

const options = computed(() => {
  return [
    {
      field: "ticket_type",
      label: "Ticket type",
      store: useTicketTypeStore(),
    },
    {
      field: "priority",
      label: "Priority",
      store: useTicketPriorityStore(),
    },
    {
      field: "agent_group",
      label: "Team",
      store: useTeamStore(),
    },
    {
      field: "customer",
      label: "Customer",
      type: "select",
      placeholder: "Select Customer",
    },
    {
      field: "subject",
      label: "Subject",
      type: "textarea",
      placeholder: "Problem in XYZ",
    },
    {
      field: "description",
      label: "Description",
      placeholder: "Problem in XYZ",
    },
  ];
});

const customFields = computed(() => {
  const _custom_fields = props.ticket.template.fields.filter(
    (f) => ["subject", "team", "priority"].indexOf(f.fieldname) === -1
  );

  return _custom_fields;
});

function update(field: Field["fieldname"], value: FieldValue, event = null) {
  if (field === "subject" && value === "") {
    createToast({
      title: "Subject is required",
      icon: "x",
      iconClasses: "text-red-600",
    });
    event.target.value = props.ticket.subject;
    return;
  }
  emit("update", { field, value });
}
</script>

<style scoped>
/* Your existing styles */
</style>
