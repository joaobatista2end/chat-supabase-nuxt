<template>
  <div class="max-w-screen-lg mx-auto">
    <div class="border-2 bg-zinc-800 text-white rounded-md p-4 mb-8 space-y-2">
      <h3 class="text-xl mb-8">Message List</h3>
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
        class="px-4 py-2 placeholder:text-zinc-500 bg-zinc-800 text-white outline-none rounded-md mb-4"
      />
    </div>
    <textarea
      v-model="state.message"
      placeholder="Type a message"
      @keyup.enter.prevent="sendMessage"
      class="block w-full h-36 px-4 py-2 placeholder:text-zinc-500 bg-zinc-800 text-white outline-none rounded-md mb-4"
    />
    <div class="flex justify-end">
      <button
        class="bg-zinc-800 text-white px-4 py-2 rounded-md"
        @click="sendMessage()"
        :disabled="pending"
      >
        {{ pending ? "Waiting" : "Send Message" }}
      </button>
    </div>
  </div>
</template>

<script lang="ts" setup>
const client = useSupabaseClient();
const messages = reactive<any[]>([]);
const randomColor = () => "#" + ((Math.random() * 0xffffff) << 0).toString(16);
const state = reactive({
  message: "",
  nick: useId(),
  color: randomColor(),
});

client
  .channel("chat_message")
  .on("broadcast", { event: "chat_message" }, (payload) => {
    messages.push(payload.payload);
  })
  .subscribe();

const { pending, execute } = useAsyncData(async () => {
  const time = useDateFormat(useNow(), "HH:mm:ss").value;
  console.log({ time });
  await client.channel("chat_message").send({
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
