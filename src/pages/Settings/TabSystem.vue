<template>
  <div>
    <div class="flex items-center">
      <svg class="h-12" viewBox="0 0 40 48" xmlns="http://www.w3.org/2000/svg">
        <path
          d="M37.73 0c1.097 0 1.986.89 1.986 1.986v43.688c0 1.096-.889 1.986-1.986 1.986H1.986A1.986 1.986 0 010 45.674V1.986C0 .889.89 0 1.986 0zm-7.943 27.404c-2.283 1.688-6.156 2.383-9.929 2.383-3.773 0-7.645-.695-9.929-2.383v4.369l.006.156c.196 2.575 5.25 3.816 9.923 3.816 4.766 0 9.93-1.291 9.93-3.972zm0-7.943c-2.283 1.688-6.156 2.383-9.929 2.383-3.773 0-7.645-.695-9.929-2.383v4.369l.006.156c.196 2.575 5.25 3.815 9.923 3.815 4.766 0 9.93-1.29 9.93-3.971zm-9.929-7.546c-4.766 0-9.929 1.29-9.929 3.972 0 2.68 5.163 3.971 9.93 3.971 4.765 0 9.928-1.29 9.928-3.971s-5.163-3.972-9.929-3.972z"
          fill="#2490EF"
          fill-rule="evenodd"
        />
      </svg>
      <div class="ml-4 flex flex-col">
        <span class="font-semibold">{{ companyName }}</span>
        <span class="text-xs text-gray-600">{{ dbPath }}</span>
      </div>
    </div>
    <div class="mt-4 flex text-sm">
      <Button @click="newDatabase">{{ _('New File') }}</Button>
      <Button class="ml-2" @click="existingDatabase">
        {{ _('Change File') }}
      </Button>
    </div>
    <p class="-ml-3 mt-4 flex items-start text-sm text-gray-600">
      <feather-icon name="info" class="mr-1 w-4 h-4 inline" />
      <!-- prettier-ignore -->
      {{ _('Create a new file to start accounting for a new company, or change file to open accounting for an existing company.') }}
    </p>
    <TwoColumnForm
      class="mt-6"
      v-if="doc"
      :doc="doc"
      :fields="fields"
      :autosave="true"
    />
    <div class="mt-6">
      <FormControl
        :df="AccountingSettings.meta.getField('autoUpdate')"
        @change="value => AccountingSettings.update('autoUpdate', value)"
        :value="AccountingSettings.autoUpdate"
      />
      <p class="pl-6 mt-1 text-sm text-gray-600">
        <!-- prettier-ignore -->
        {{ _('Automatically check for updates and download them if available. The update will be applied after you restart the app.') }}
      </p>
    </div>
  </div>
</template>

<script>
import frappe from 'frappejs';
import TwoColumnForm from '@/components/TwoColumnForm';
import FormControl from '@/components/Controls/FormControl';
import Button from '@/components/Button';
import { createNewDatabase, loadExistingDatabase } from '@/utils';
import { remote } from 'electron';

export default {
  name: 'TabSystem',
  components: {
    TwoColumnForm,
    FormControl,
    Button
  },
  data() {
    return {
      companyName: null,
      doc: null
    };
  },
  async mounted() {
    this.doc = frappe.SystemSettings;
    this.companyName = frappe.AccountingSettings.companyName;
  },
  methods: {
    async newDatabase() {
      let filePath = await createNewDatabase();
      localStorage.dbPath = filePath;
      frappe.events.trigger('reload-main-window');
      remote.getCurrentWindow().close();
    },
    async existingDatabase() {
      let filePath = await loadExistingDatabase();
      localStorage.dbPath = filePath;
      frappe.events.trigger('reload-main-window');
      remote.getCurrentWindow().close();
    }
  },
  computed: {
    fields() {
      let meta = frappe.getMeta('SystemSettings');
      return meta.getQuickEditFields();
    },
    dbPath() {
      return localStorage.dbPath;
    },
    AccountingSettings() {
      return frappe.AccountingSettings;
    }
  }
};
</script>
