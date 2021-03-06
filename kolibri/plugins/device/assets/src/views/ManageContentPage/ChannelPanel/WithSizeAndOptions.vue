<template>

  <div
    class="panel"
    :class="{'panel-sm': windowIsSmall}"
    :style="{ borderTop: `1px solid ${$themePalette.grey.v_200}` }"
  >
    <ChannelDetails :channel="channel" />

    <div
      class="col-2"
      dir="auto"
      data-test="resources-size"
    >
      {{ resourcesSizeText }}
    </div>

    <div class="col-3">
      <KDropdownMenu
        :text="coreString('optionsLabel')"
        :disabled="disabled"
        :options="dropdownOptions"
        :position="dropdownPosition"
        @select="handleManageChannelAction($event.value)"
      />
    </div>
  </div>

</template>


<script>

  // ChannelPanel with Details, On-Device Size, and Options Dropdown

  import responsiveWindowMixin from 'kolibri.coreVue.mixins.responsiveWindowMixin';
  import commonCoreStrings from 'kolibri.coreVue.mixins.commonCoreStrings';
  import bytesForHumans from 'kolibri.utils.bytesForHumans';
  import ChannelDetails from './ChannelDetails';

  export default {
    name: 'WithSizeAndOptions',
    components: {
      ChannelDetails,
    },
    mixins: [responsiveWindowMixin, commonCoreStrings],
    props: {
      channel: {
        type: Object,
        required: true,
      },
      disabled: {
        type: Boolean,
        default: false,
      },
    },
    computed: {
      resourcesSizeText() {
        return bytesForHumans(this.channel.on_device_file_size);
      },
      dropdownPosition() {
        // On small screens, position to the left so it doesn't overflow the
        // window and mess up the global layout.
        return this.windowIsSmall ? 'bottom left' : 'bottom right';
      },
      dropdownOptions() {
        return [
          {
            label: this.$tr('manageChannelAction'),
            value: 'MANAGE_CHANNEL',
          },
          {
            label: this.$tr('deleteChannelAction'),
            value: 'DELETE_CHANNEL',
          },
        ];
      },
    },
    methods: {
      handleManageChannelAction(action) {
        if (action === 'DELETE_CHANNEL') {
          return this.$emit('select_delete');
        }
        return this.$emit('select_manage', { ...this.channel });
      },
    },
    $trs: {
      manageChannelAction: {
        message: 'Manage',
        context: '\nOperation that can be performed on a channel',
      },
      deleteChannelAction: 'Delete',
    },
  };

</script>


<style lang="scss" scoped>

  .panel {
    display: flex;
    padding: 32px 0;
  }

  .panel-sm {
    flex-direction: column;
    padding: 16px 0;
  }

  .col-2 {
    min-width: 80px;
    margin-right: 16px;
    text-align: right;

    .panel-sm & {
      font-size: 14px;
      text-align: left;
    }
  }

  .col-3 {
    margin-top: -8px;

    .panel-sm & {
      margin-top: 16px;
      text-align: right;
    }
  }

</style>
