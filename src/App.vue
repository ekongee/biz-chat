<!-- App.vue -->
<template>
  <v-app>
    <v-navigation-drawer
      v-model="drawer"
      app
      theme="dark"
      class="bg-primary-darken-1"
      :width="280"
    >
      <!-- 사용자 프로필 -->
      <v-list-item class="px-3 py-2">
        <template v-slot:prepend>
          <v-avatar color="blue" size="40">
            <span class="text-white">홍</span>
          </v-avatar>
        </template>
        <v-list-item-title class="font-weight-medium">홍길동</v-list-item-title>
        <v-list-item-subtitle class="text-green-lighten-2">온라인</v-list-item-subtitle>
        <template v-slot:append>
          <v-btn icon="mdi-bell" variant="text" size="small"></v-btn>
        </template>
      </v-list-item>

      <v-divider></v-divider>

      <!-- 채널 목록 -->
      <v-list nav density="compact" class="py-0">
        <v-list-subheader class="text-uppercase text-grey-lighten-1">
          채널
          <v-spacer></v-spacer>
          <v-btn icon="mdi-plus" variant="text" size="x-small" class="text-grey-lighten-1"></v-btn>
        </v-list-subheader>

        <v-list-item
          v-for="channel in channels"
          :key="channel.id"
          :value="channel.id"
          :active="selectedChat === channel.id"
          active-color="blue-lighten-2"
          @click="selectedChat = channel.id"
        >
          <template v-slot:prepend>
            <v-icon size="small">mdi-pound</v-icon>
          </template>
          <v-list-item-title>{{ channel.name }}</v-list-item-title>
          <template v-slot:append v-if="channel.unread > 0">
            <v-badge
              :content="channel.unread"
              color="red"
              offset-x="2"
              offset-y="2"
            ></v-badge>
          </template>
        </v-list-item>

        <!-- 다이렉트 메시지 -->
        <v-list-subheader class="text-uppercase text-grey-lighten-1 mt-3">
          다이렉트 메시지
          <v-spacer></v-spacer>
          <v-btn icon="mdi-plus" variant="text" size="x-small" class="text-grey-lighten-1"></v-btn>
        </v-list-subheader>

        <v-list-item
          v-for="user in directMessages"
          :key="user.id"
          :value="user.id"
          :active="selectedChat === user.id"
          active-color="blue-lighten-2"
          @click="selectedChat = user.id"
        >
          <template v-slot:prepend>
            <v-badge
              :color="user.status === 'online' ? 'green' : user.status === 'away' ? 'amber' : 'grey'"
              dot
              location="bottom end"
              offset-x="3"
              offset-y="3"
            >
              <v-avatar size="24" color="grey-lighten-1">
                <span class="text-white text-caption">{{ user.name.charAt(0) }}</span>
              </v-avatar>
            </v-badge>
          </template>
          <v-list-item-title>{{ user.name }}</v-list-item-title>
        </v-list-item>

        <!-- 앱 통합 -->
        <v-list-subheader class="text-uppercase text-grey-lighten-1 mt-3">
          앱
        </v-list-subheader>

        <v-list-item>
          <template v-slot:prepend>
            <v-icon size="small">mdi-calendar</v-icon>
          </template>
          <v-list-item-title>캘린더</v-list-item-title>
        </v-list-item>

        <v-list-item>
          <template v-slot:prepend>
            <v-icon size="small">mdi-file-document</v-icon>
          </template>
          <v-list-item-title>문서</v-list-item-title>
        </v-list-item>

        <v-list-item>
          <template v-slot:prepend>
            <v-icon size="small">mdi-checkbox-marked-outline</v-icon>
          </template>
          <v-list-item-title>할 일</v-list-item-title>
        </v-list-item>

        <v-list-item>
          <template v-slot:prepend>
            <v-icon size="small">mdi-video</v-icon>
          </template>
          <v-list-item-title>화상 회의</v-list-item-title>
        </v-list-item>

        <v-divider class="my-3"></v-divider>

        <v-list-item>
          <template v-slot:prepend>
            <v-icon size="small">mdi-cog</v-icon>
          </template>
          <v-list-item-title>설정</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <!-- Main Chat Area -->
    <v-main>
      <!-- Chat Header -->
      <v-app-bar
        flat
        elevation="1"
        height="64"
        color="white"
      >
        <template v-slot:prepend>
          <v-app-bar-nav-icon
            class="d-md-none"
            @click="drawer = !drawer"
          ></v-app-bar-nav-icon>
          <v-icon class="mr-2">mdi-pound</v-icon>
        </template>

        <v-toolbar-title class="font-weight-medium">
          팀-일반
          <div class="text-caption text-secondary">
            멤버 15 · 온라인 8
          </div>
        </v-toolbar-title>

        <v-spacer></v-spacer>

        <v-text-field
          prepend-inner-icon="mdi-magnify"
          hide-details
          label="검색"
          class="mx-2 mt-5"
          variant="outlined"
          density="compact"
          style="max-width: 240px;"
        ></v-text-field>

        <v-btn icon class="ml-2">
          <v-icon>mdi-video</v-icon>
        </v-btn>

        <v-btn icon class="ml-2">
          <v-icon>mdi-account-group</v-icon>
        </v-btn>

        <v-btn
          icon
          class="d-none d-lg-flex ml-2"
          @click="rightDrawer = !rightDrawer"
        >
          <v-icon>{{ rightDrawer ? 'mdi-chevron-right' : 'mdi-chevron-left' }}</v-icon>
        </v-btn>
      </v-app-bar>

      <!-- Chat Content -->
      <div style="height: calc(100vh - 64px); display: flex; flex-direction: column;">
        <!-- Messages Container -->
        <v-sheet class="flex-grow-1 overflow-y-auto pa-4">
          <!-- Pinned Message -->
          <v-alert
            color="amber-lighten-5"
            border="start"
            border-color="amber"
            icon="mdi-pin"
            class="body-2"
            density="compact"
          >
            <div class="d-flex align-center justify-space-between">
              <div>
                <span class="font-weight-medium">박지훈:</span> 
                오후 2시 화상 회의 가능하신가요?
              </div>
              <v-btn icon="mdi-close" variant="text" size="x-small"></v-btn>
            </div>
          </v-alert>

          <!-- Date Separator -->
          <div class="text-center my-6">
            <v-chip
              size="small"
              class="bg-grey-lighten-3"
            >
              오늘, 4월 29일
            </v-chip>
          </div>

          <!-- Chat Messages -->
          <div
            v-for="message in messages"
            :key="message.id"
            :class="['mb-4', message.sender === '나' ? 'text-right' : '']"
          >
            <div
              v-if="message.sender !== '나'"
              class="d-flex align-start"
            >
              <v-avatar
                size="36"
                color="grey-lighten-2"
                class="mr-3"
              >
                <span class="body-2">{{ message.sender.charAt(0) }}</span>
              </v-avatar>
              <div>
                <div class="d-flex align-center mb-1">
                  <span class="font-weight-medium">{{ message.sender }}</span>
                  <span class="text-caption ml-2 text-grey">{{ message.time }}</span>
                  <div class="ml-2 d-flex align-center message-actions">
                    <v-btn icon="mdi-star-outline" variant="text" size="x-small" class="mr-1"></v-btn>
                    <v-btn icon="mdi-pin" variant="text" size="x-small" class="mr-1"></v-btn>
                    <v-btn icon="mdi-dots-horizontal" variant="text" size="x-small"></v-btn>
                  </div>
                </div>
                <v-card
                  variant="outlined"
                  class="pa-3 rounded-lg bg-grey-lighten-4"
                  style="display: inline-block; max-width: 80%;"
                >
                  <div>{{ message.content }}</div>
                  <v-card
                    v-if="message.attachments"
                    v-for="file in message.attachments"
                    :key="file"
                    variant="outlined"
                    class="mt-2 pa-2 d-flex align-center"
                  >
                    <v-icon
                      size="small"
                      color="blue"
                      class="mr-2"
                    >
                      mdi-file-document
                    </v-icon>
                    <span class="text-caption">{{ file }}</span>
                  </v-card>
                </v-card>
                <div class="mt-1">
                  <v-btn
                    variant="text"
                    size="x-small"
                    class="text-caption mr-2 px-1"
                  >
                    답장
                  </v-btn>
                  <v-btn
                    variant="text"
                    size="x-small"
                    class="text-caption px-1"
                  >
                    쓰레드
                  </v-btn>
                </div>
              </div>
            </div>

            <div v-else>
              <div class="d-flex align-center justify-end mb-1">
                <div class="mr-2 d-flex align-center message-actions">
                  <v-btn icon="mdi-dots-horizontal" variant="text" size="x-small"></v-btn>
                  <v-btn icon="mdi-pin" variant="text" size="x-small" class="mx-1"></v-btn>
                  <v-btn icon="mdi-star-outline" variant="text" size="x-small"></v-btn>
                </div>
                <span class="text-caption text-grey">{{ message.time }}</span>
              </div>
              <v-card
                color="primary"
                class="pa-3 rounded-lg text-white"
                style="display: inline-block; max-width: 80%;"
              >
                <div>{{ message.content }}</div>
              </v-card>
            </div>
          </div>
        </v-sheet>

        <!-- Message Input -->
        <v-sheet
          class="pt-2 pb-4 px-4"
          elevation="3"
        >
          <v-card variant="outlined">
            <v-card-text class="py-2">
              <div class="d-flex">
                <v-btn icon="mdi-paperclip" variant="text" class="mr-1"></v-btn>
                <v-text-field
                  placeholder="메시지 작성..."
                  hide-details
                  variant="outlined"
                  density="compact"
                  class="flex-grow-1"
                ></v-text-field>
                <v-btn icon="mdi-clock-outline" variant="text" class="mx-1"></v-btn>
                <v-btn color="primary" icon="mdi-send" variant="text"></v-btn>
              </div>
            </v-card-text>
          </v-card>
          <div class="d-flex justify-space-between text-caption mt-1 text-grey">
            <div>
              <span class="mx-1 text-blue">@멘션</span>
              <span class="mx-1 text-blue">#채널</span>
              <span class="mx-1 text-blue">:이모지:</span>
            </div>
            <div>
              <span>Enter로 전송, Shift+Enter로 줄바꿈</span>
            </div>
          </div>
        </v-sheet>
      </div>
    </v-main>

    <!-- Right Sidebar -->
    <v-navigation-drawer
      v-model="rightDrawer"
      location="right"
      width="320"
      class="bg-grey-lighten-4"
      floating
    >
      <v-list>
        <v-list-item>
          <v-list-item-title class="text-h6 font-weight-medium">할 일</v-list-item-title>
        </v-list-item>

        <v-list-item class="px-2">
          <v-card
            variant="outlined"
            class="pa-3 mb-2 rounded w-100"
          >
            <div class="d-flex">
              <v-checkbox
                class="mt-0 mr-2"
                hide-details
              ></v-checkbox>
              <div>
                <div class="font-weight-medium">분기 보고서 검토</div>
                <div class="text-caption text-grey">마감: 오늘 오후 5시</div>
              </div>
            </div>
          </v-card>
        </v-list-item>

        <v-list-item class="px-2">
          <v-card
            variant="outlined"
            class="pa-3 mb-2 rounded w-100"
          >
            <div class="d-flex">
              <v-checkbox
                class="mt-0 mr-2"
                hide-details
              ></v-checkbox>
              <div>
                <div class="font-weight-medium">고객 미팅 준비</div>
                <div class="text-caption text-grey">마감: 내일 오전 10시</div>
              </div>
            </div>
          </v-card>
        </v-list-item>

        <v-list-item>
          <v-btn
            variant="text"
            size="small"
            color="primary"
            class="pl-2"
            prepend-icon="mdi-plus"
          >
            할 일 추가
          </v-btn>
        </v-list-item>

        <v-divider class="my-2"></v-divider>

        <v-list-item>
          <v-list-item-title class="text-h6 font-weight-medium">다가오는 일정</v-list-item-title>
        </v-list-item>

        <v-list-item class="px-2">
          <v-card
            variant="outlined"
            class="pa-3 mb-2 rounded w-100"
          >
            <div class="font-weight-medium">팀 주간 회의</div>
            <div class="text-caption text-grey">오늘 오후 2시 - 3시</div>
          </v-card>
        </v-list-item>

        <v-list-item class="px-2">
          <v-card
            variant="outlined"
            class="pa-3 mb-2 rounded w-100"
          >
            <div class="font-weight-medium">프로젝트 A 미팅</div>
            <div class="text-caption text-grey">내일 오전 11시 - 12시</div>
          </v-card>
        </v-list-item>

        <v-divider class="my-2"></v-divider>

        <v-list-item>
          <v-list-item-title class="text-h6 font-weight-medium">공유된 파일</v-list-item-title>
        </v-list-item>

        <v-list-item class="px-2">
          <v-card
            variant="outlined"
            class="pa-3 mb-2 rounded w-100"
          >
            <div class="d-flex">
              <v-icon
                size="small"
                color="blue"
                class="mr-2 mt-1"
              >
                mdi-file-document
              </v-icon>
              <div>
                <div class="font-weight-medium">분기_보고서.pdf</div>
                <div class="text-caption text-grey">김민준, 오늘 오전 9:30</div>
              </div>
            </div>
          </v-card>
        </v-list-item>

        <v-list-item class="px-2">
          <v-card
            variant="outlined"
            class="pa-3 rounded w-100"
          >
            <div class="d-flex">
              <v-icon
                size="small"
                color="blue"
                class="mr-2 mt-1"
              >
                mdi-file-document
              </v-icon>
              <div>
                <div class="font-weight-medium">마케팅_전략.pptx</div>
                <div class="text-caption text-grey">이서연, 어제</div>
              </div>
            </div>
          </v-card>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
  </v-app>
</template>

<script setup>
import { ref } from 'vue';

const drawer = ref(true);
const rightDrawer = ref(true);
const selectedChat = ref('team-general');

const channels = ref([
  { id: 'team-general', name: '팀-일반', unread: 5 },
  { id: 'project-a', name: '프로젝트A', unread: 0 },
  { id: 'marketing', name: '마케팅', unread: 12 },
]);

const directMessages = ref([
  { id: 'user1', name: '김민준', status: 'online' },
  { id: 'user2', name: '이서연', status: 'offline' },
  { id: 'user3', name: '박지훈', status: 'away' },
  { id: 'user4', name: '최지훈', status: 'online' },
  { id: 'user5', name: '정수빈', status: 'away' }, 
  { id: 'user6', name: '정하늘', status: 'online' },
  { id: 'user7', name: '이도현', status: 'offline' },
  { id: 'user8', name: '김수현', status: 'away' },
  { id: 'user9', name: '박서준', status: 'online' },
  { id: 'user10', name: '최유리', status: 'offline' },
  
  { id: 'user11', name: '이재훈', status: 'online' },
  { id: 'user12', name: '김지민', status: 'offline' },
  { id: 'user13', name: '박상현', status: 'away' },
  { id: 'user14', name: '최민수', status: 'online' },
  { id: 'user15', name: '정하은', status: 'offline' },
]);

const messages = ref([
  { id: 1, sender: '김민준', time: '오전 9:30', content: '오늘 회의 자료 공유드립니다.', attachments: ['분기_보고서.pdf'] },
  { id: 2, sender: '이서연', time: '오전 9:45', content: '감사합니다. 검토 후 의견 드리겠습니다.' },
  { id: 3, sender: '박지훈', time: '오전 10:15', content: '오후 2시 화상 회의 가능하신가요?', isPinned: true },
  { id: 4, sender: '나', time: '오전 10:20', content: '네, 가능합니다. 회의실 예약해 두겠습니다.' },
  { id: 5, sender: '박지훈', time: '오전 10:25', content: '감사합니다.', isPinned: true },
  
  { id: 6, sender: '나', time: '오전 10:30', content: '회의 링크는 다음과 같습니다.', attachments: ['회의_링크.url'] },
  { id: 7, sender: '김민준', time: '오전 10:35', content: '확인했습니다. 감사합니다.' },
  { id: 8, sender: '이서연', time: '오전 10:40', content: '저도 확인했습니다.' },
  { id: 9, sender: '박지훈', time: '오전 10:45', content: '회의 준비 잘 부탁드립니다.' },
  { id: 10, sender: '나', time: '오전 10:50', content: '네, 알겠습니다.' },
]);
</script>

<style>
.message-actions {
  opacity: 0;
  transition: opacity 0.2s ease;
}

.message-actions:hover {
  opacity: 1;
}
</style>