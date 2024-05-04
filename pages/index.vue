<template>
  <div class="min-h-screen bg-zinc-800">
    <div class="max-w-[460px] mx-auto py-10">
      <div class="text-zinc-100">
        <h3 class="text-3xl font-bold text-center mb-8">Nuxt Supabase Chat</h3>
        <p>
          Nuxt Supabase Chat é uma aplicação de mensagens simples e rápida
          projetada para facilitar a comunicação instantânea entre amigos,
          colegas e grupos. Simples, eficiente e direto ao ponto!
        </p>

        <div class="mt-8 flex justify-center">
          <div>
            <input
              type="text"
              v-model="channelName"
              placeholder="Insira o canal"
              class="px-4 py-2 placeholder:text-zinc-500 bg-white text-zinc-800 outline-none rounded-l mb-4"
              @keyup.enter="redirectToChannel"
            />
            <button
              @click="redirectToChannel"
              class="hover:bg-green-600 transition-all bg-green-500 text-white px-4 py-2 font-semibold rounded-r"
            >
              Iniciar chat
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
const channelName = ref<string>();
const router = useRouter();

const redirectToChannel = () => {
  router.push(`channel/${channelName.value}`);
}
</script>
