<template>
  <div class="mx-auto max-w-5xl">
    <div class="flex flex-col space-y-5 h-screen pt-5 pb-5">
      <div class="box bg-slate-100 flex justify-between">
        <button
            @click="resetAll"
            class="flex items-center bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-md p-2 hover:border-gray-500 shadow-sm"
        >
          New chat
          <PencilSquareIcon class="w-4 h-4 ml-2"/>
        </button>
        <ModelSelector v-model="selectedModel"/>
      </div>
      <div class="grow overflow-y-auto" id="chat-box">
        <div
            v-if="chatContext.length == 0"
            class="flex flex-col justify-center box bg-slate-100 min-h-full"
        >
          <h1 class="text-center font-bold text-2xl text-indigo-600 light-blue">
            How are you feeling today?
          </h1>

          <div class="grid grid-cols-3 gap-5 mt-10">
            <div
                class="border border-gray-300 p-5 rounded-lg cursor-pointer hover:border-gray-600"
                v-for="item in helpTextList"
                @click="handleQuickPrompt(item)"
            >
              {{ item }}
            </div>
          </div>
        </div>

        <div class="space-y-2" v-else>
          <div v-for="chat in chatContext">
            <div v-if="chat.role === 'user'" :class="chat.role">
              <markdown-render :class="chat.role+'-content'" :content="chat.content"/>
             <div>
                <h1 :class="chat.role+'-title'"></h1>
             </div>
            </div>
            <div v-else :class="chat.role">
              <div>
                 <h1 :class="chat.role+'-title'"></h1>
              </div>
              <markdown-render :content="chat.content" :class="chat.role+'-content'"/>
            </div>
          </div>

          <div class="box" v-if="loadingAnswer || aiAnswer">
            <div v-if="loadingAnswer">
              <div class="flex">
                <h1 class="font-black">Wahoo</h1>
                <span>
                  <svg
                      aria-hidden="true"
                      class="w-4 h-4 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600 ml-2 mt-1"
                      viewBox="0 0 100 101"
                      fill="none"
                      xmlns="http://www.w3.org/2000/svg"
                  >
                    <path
                        d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
                        fill="currentColor"
                    />
                    <path
                        d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
                        fill="currentFill"
                    />
                  </svg>
                </span>
              </div>
            </div>
            <div v-else v-if="aiAnswer">
              <h1 class="font-bold">Wahoo</h1>
              <markdown-render :content="aiAnswer"/>
            </div>
          </div>
        </div>
      </div>
      <div>
        <prompter v-model="prompt" @submit="handleSend"/>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import {useChatContext} from "./composables/useChatPrompt";
import type IChatContext from "./models/IChatContext";
import {PencilSquareIcon} from "@heroicons/vue/24/solid";
import axios from 'axios';

const {
  chatContext,
  addChatContextItem,
  prompt,
  getContextToString,
  resetPrompt,
  aiAnswer,
  loadingAnswer,
  toggleLoading,
  resetAll,
  selectedModel,
} = useChatContext<IChatContext>();

const helpTextList = reactive([
  "How to avoid feeling inferior.",
  "I feel very depressed today.",
  "I always feel like there's a voice talking to me.",
  "I lack motivation for anything.",
  "My relationship with my family is strained.",
  "I need ways to reduce my anxiety.",
  "I’m not satisfied with myself.",
  "I’m constantly anxious.",
  "Is being sentimental a disease?",
]);

const handleSend = async () => {
  if (loadingAnswer.value) return;

  toggleLoading();

  addChatContextItem({
    role: "user",
    content: prompt.value,
  });


  fetch(`/api/chat?model=${selectedModel.value}`, {
    method: "post",
    body: getContextToString(),
  }).then((response) => {
    toggleLoading();
    resetPrompt();
    const reader = response.body?.getReader();
    return reader?.read().then(function read(result): any {
      if (result.done) {
        addChatContextItem({
          role: "assistant",
          content: aiAnswer.value,
        });

        return;
      }

      aiAnswer.value += new TextDecoder().decode(result.value);

      chatBoxScrollBottom();
      return reader.read().then(read);
    });
  });
};

const chatBoxScrollBottom = () => {
  const chatBox = document.getElementById("chat-box");
  if (chatBox) {
    chatBox.scrollTop = chatBox.scrollHeight;
  }
};


const handleQuickPrompt = (qprompt: string) => {
  prompt.value = qprompt;
  handleSend();
};
</script>
