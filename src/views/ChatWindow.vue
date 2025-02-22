<template>
  <div class="chatarea">
    <div class="card">
      <PanelMenu :model="state.serverMenuOptions" style="margin-top:18px;"/>
      <div class="virtualscroller-demo">
        <div class="p-d-flex p-dir-col p-mr-3 p-mt-3">
          <VirtualScroller :items="state.availableUsers" :itemSize="state.availableUsers.length-1">
            <template v-slot:item="{ item }" >
              <div style="background-color:lightgray;border-radius:2px;padding:5px; margin-top:15px; display:flex;align-content:center;">
                <div
                @click="privateChat(item.username)"
              >
                <Avatar
                  v-if="item.profile_uri === ''"
                  :label="
                    item.first_name.charAt(0).toUpperCase() +
                      item.last_name.charAt(0).toUpperCase()
                  "
                  style="color:black;user-select:none;"
                />
                <Avatar
                  v-else
                  :image="item.profile_uri"
                />
                {{ item.username }}
              </div>
              </div>
            </template>
          </VirtualScroller>
        </div>
      </div>
    </div>
    <div v-if="state.isMounted" class="chat-window">
      <Chat :serverName="state.server_code" :room="state.server_room" :tenants="property.tenants" />
    </div>
  </div>
</template>

<script>
import { toRefs, onMounted, reactive } from 'vue';
import { useStore } from 'vuex';

import Chat from '@/components/Chat.vue';
export default {
  name: 'ChatWindow',
  setup(props) {
    const store = useStore();
    const { server_code } = toRefs(props);
    const property = store.getters.getCurrentUserProperties.find(
      (p) => p.server_code == server_code.value,
    );
    const state = reactive({
      isMounted: false,
      server_code: server_code,
      availableUsers: property.tenants.filter(
        (t) =>
          t.user_status === 0 &&
          t.username !== store.getters.getCurrentUser.username,
      ),
      server_room: 'general',
      serverMenuOptions: property.channels.map((p) => {
        return {
          label: p,
          icon: 'pi pi-check',
          command: () => {
            state.server_room = p;
          },
        };
      }),
    });

    onMounted(() => (state.isMounted = true));
    const privateChat = (username) => state.server_room = `${store.getters.getCurrentUser.username}:${username}`

    return { state, privateChat, property };
  },
  components: {
    Chat,
  },
  props: {
    server_code: {
      type: String,
    },
  },
};
</script>

<style scoped lang="scss">
.p-panelmenu {
  width: 22rem;
}

.chat-window {
  margin: 20px;
}

.chatarea {
  display: flex;
}

.virtualscroller-demo {
  background-color: transparent;
  margin-top: 15px;
  ::v-deep(.p-virtualscroller) {
    width: auto;
    height: 250px;
    

    .scroll-item {
      background-color: lightgray;
      display: flex;
      align-items: center;
    }

    .custom-scroll-item {
      background-color: lightgray;
      flex-direction: column;
      align-items: stretch;
    }

    // .odd {
    //   background-color: var(--surface-b);
    // }
  }

  ::v-deep(.p-horizontal-scroll) {
    .p-virtualscroller-content {
      background-color: lightgray;
      display: flex;
      flex-direction: row;
    }

    .scroll-item {
      writing-mode: vertical-lr;
    }
  }

  ::v-deep(.custom-loading > .p-virtualscroller-loader) {
    display: block;
  }
}
</style>
