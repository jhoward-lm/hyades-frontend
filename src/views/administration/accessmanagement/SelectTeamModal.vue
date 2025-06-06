<template>
  <b-modal
    id="selectTeamModal"
    size="lg"
    hide-header-close
    no-stacking
    :title="$t('admin.select_team')"
  >
    <bootstrap-table
      ref="table"
      :columns="columns"
      :data="data"
      :options="options"
    >
    </bootstrap-table>
    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="secondary" @click="cancel()">{{
        $t('message.cancel')
      }}</b-button>
      <b-button
        size="md"
        variant="primary"
        :disabled="selectionHasChanged"
        @click="handleSelection"
        >{{ $t('message.select') }}</b-button
      >
    </template>
  </b-modal>
</template>

<script>
import xssFilters from 'xss-filters';
import permissionsMixin from '../../../mixins/permissionsMixin';
import common from '../../../shared/common';

export default {
  props: {
    currentTeams: {
      type: Array,
      default: () => [],
    },
  },
  mixins: [permissionsMixin],
  data() {
    return {
      currentSelection: [],
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
      columns: [
        {
          field: 'state',
          checkbox: true,
          align: 'center',
        },
        {
          title: this.$t('admin.team_name'),
          field: 'name',
          sortable: true,
          formatter(value) {
            return xssFilters.inHTMLData(common.valueWithDefault(value, ''));
          },
        },
      ],
      data: [],
      options: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        silentSort: false,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        icons: {
          refresh: 'fa-refresh',
        },
        responseHandler: function (res, xhr) {
          res.total = xhr.getResponseHeader('X-Total-Count');
          return res;
        },
        url: `${this.$api.BASE_URL}/${this.$api.URL_TEAM}`,
        onLoadSuccess: () => {
          const preSelected = this.currentTeams.map((team) => team.name);
          this.$refs.table.checkBy({
            field: 'name',
            values: preSelected,
          });
        },
        onCheck: this.updateCurrentSelection,
        onUncheck: this.updateCurrentSelection,
        onCheckAll: this.updateCurrentSelection,
        onUncheckAll: this.updateCurrentSelection,
      },
    };
  },
  computed: {
    selectionHasChanged() {
      if (
        this.currentSelection.length === 0 &&
        this.currentTeams.length === 0
      ) {
        return true;
      }

      if (this.currentSelection.length !== this.currentTeams.length) {
        return false;
      }

      const isEqual = this.currentSelection.every((sel) =>
        this.currentTeams.some((team) => team.name === sel.name),
      );

      return (
        isEqual &&
        this.currentTeams.every((team) =>
          this.currentSelection.some((sel) => sel.name === team.name),
        )
      );
    },
  },
  methods: {
    handleSelection: function () {
      // this.$root.$emit('bv::hide::modal', this.$children[0].id);
      this.$bvModal.hide('selectTeamModal');
      this.$emit('selection', this.currentSelection);
    },
    updateCurrentSelection() {
      this.currentSelection = this.$refs.table.getSelections();
    },
  },
};
</script>
