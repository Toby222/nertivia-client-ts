<template>
  <ContextMenu
    ref="context"
    :items="items"
    :pos="pos"
    @itemClick="itemClick"
    @close="close"
  />
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import ContextMenu from "@/components/ContextMenu.vue";
import { PopoutsModule } from "@/store/modules/popouts";
import { ServersModule } from "@/store/modules/servers";
import { MeModule } from "@/store/modules/me";
import router from "@/router";
import { ServerMembersModule } from "@/store/modules/serverMembers";
import { NotificationsModule } from "@/store/modules/notifications";
import { LastSeenServerChannelsModule } from "@/store/modules/lastSeenServerChannel";
import { ChannelsModule } from "@/store/modules/channels";
import { warn } from "vue-class-component/lib/util";
import { MutedChannelsModule } from "@/store/modules/mutedChannels";
import {
  muteServerChannel,
  unmuteServerChannel
} from "@/services/serverService";

@Component({ components: { ContextMenu } })
export default class extends Vue {
  @Prop() private data!: {
    x: number;
    y: number;
    server_id: string;
    channelID: string;
  };

  close() {
    PopoutsModule.ClosePopout("context");
  }
  itemClick(item: any) {
    switch (item.name) {
      case "Manage Notification":
        this.$router.push(
          `/app/servers/${this.data.server_id}/settings/manage-notification`
        );
        break;
      case "Copy ID":
        this.$copyText(this.data.channelID);
        break;
      case "Mark As Read":
        this.markAsRead();
        break;
      case "Unmute Channel":
        unmuteServerChannel(this.server.server_id, this.channel.channelID);
        break;
      case "Mute Channel":
        muteServerChannel(this.server.server_id, this.channel.channelID);
        break;
      default:
        break;
    }
  }
  markAsRead() {
    this.$socket.client.emit("notification:dismiss", {
      channelID: this.data.channelID
    });
  }
  get items() {
    const items: any = [
      {
        name: "Mark As Read",
        icon: "markunread_mailbox",
        disabled: !this.notifications
      },
      {
        name: this.isMuted ? "Unmute Channel" : "Mute Channel",
        icon: this.isMuted ? "notifications" : "notifications_off",
        warn: !this.isMuted
      },
      { type: "seperator" },
      {
        name: "Copy ID",
        icon: "developer_board"
      }
    ];

    return items;
  }
  get pos() {
    return {
      x: this.data.x,
      y: this.data.y
    };
  }
  get server() {
    return ServersModule.servers[this.data.server_id];
  }
  get channel() {
    return ChannelsModule.channels[this.data.channelID];
  }
  get notifications() {
    return LastSeenServerChannelsModule.serverChannelNotification(
      this.data.channelID
    );
  }
  get isMuted() {
    return MutedChannelsModule.mutedChannels.includes(this.channel.channelID);
  }
}
</script>

<style scoped></style>