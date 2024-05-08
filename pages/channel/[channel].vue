<template>
  <div class="bg-zinc-800 h-screen">
    <div class="max-w-full w-[640px] mx-auto pt-10 px-8">
      <h3 class="text-xl mb-4 text-white font-bold">Canal: {{ channel }}</h3>
      <div
        class="bg-zinc-700 text-white rounded-md p-4 mb-4 space-y-2 max-h-60 h-60 overflow-scroll"
      >
        <div v-for="(message, index) in messages" :key="index">
          <div
            class="bg-zinc-700 text-sm px-2 py-1 rounded-sm inline"
            v-if="message?.message"
          >
            <div
              class="inline-flex text-xs px-2 rounded-full"
              :style="{ backgroundColor: message.color }"
            >
              {{ message?.nick }}
            </div>
            <span class="text-xs mx-2 text-zinc-300">
              {{ message?.time }}
            </span>

            {{ message?.message }}
          </div>
        </div>
      </div>

      <div>
        <label for="" class="text-zinc-800 font-medium mr-2 block"
          >My nick:
        </label>
        <input
          type="text"
          v-model="state.nick"
          class="px-4 py-2 placeholder:text-zinc-500 bg-zinc-700 text-white outline-none rounded-md mb-4"
        />
      </div>

      <textarea
        v-model="state.message"
        placeholder="Escreva uma mensagem"
        @keyup.enter.prevent="sendMessage"
        class="block w-full h-36 px-4 py-2 placeholder:text-zinc-500 bg-zinc-700 text-white outline-none rounded-md mb-4"
      />
      <div class="flex justify-end">
        <button
          :class="{ 'bg-green-500 text-white px-4 py-2 rounded-md font-semibold': !pending, 'bg-green-400 text-white px-4 py-2 rounded-md font-semibold': pending}"
          @click="sendMessage()"
          :disabled="pending"
        >
          {{ pending ? "Aguarde..." : "Enviar" }}
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { uid } from "uid";

const { params } = useRoute();
const channel = computed(() => (params?.channel as string) || id);
const client = useSupabaseClient();
const id = uid();
const messages = reactive<any[]>([]);
const randomColor = () => "#" + ((Math.random() * 0xffffff) << 0).toString(16);
const state = reactive({
  message: "",
  nick: id,
  color: randomColor(),
});

client
  .channel(channel.value)
  .on("broadcast", { event: "chat_message" }, (payload) => {
    messages.push(payload.payload);
  })
  .subscribe();

const { pending, execute } = useAsyncData(async () => {
  const time = useDateFormat(useNow(), "HH:mm:ss").value;
  await client.channel(channel.value).send({
    type: "broadcast",
    event: "chat_message",
    payload: {
      message: state.message,
      nick: state.nick,
      color: state.color,
      time,
    },
  });
});

const sendMessage = async () => {
  execute();
  state.message = "";
};
</script>
