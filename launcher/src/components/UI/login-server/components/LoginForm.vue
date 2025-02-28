<template>
  <div class="col-start-1 col-span-full row-start-1 row-span-full grid grid-cols-24 grid-rows-12">
    <div
      class="col-start-1 col-span-full row-start-1 row-span-1 bg-[#264744] flex justify-center items-center rounded-t-md"
    >
      <span class="text-2xl text-gray-300 font-bold">{{ $t("formsetup.server") }}</span>
    </div>

    <IpScanModal
      v-if="ipScanModal"
      :btn-state="btnSearchState"
      :scanned-ip="foundIp"
      @close-ipscan="ipScanModal = false"
      @btn-function="scanFunction"
    />
    <div v-if="serverStore.alertBox" class="alert animate__animated animate__flipInX">
      {{ $t("formsetup.fillFields") }}
    </div>

    <div
      v-if="serverStore.connectingAnimActive && router.currentRoute.value.name !== 'login'"
      class="col-start-1 col-span-full row-start-1 row-span-full flex flex-col justify-center items-center z-20"
    >
      <div class="w-full h-full absolute inset-0 bg-black rounded-md opacity-80"></div>
      <div class="w-full h-full flex flex-col justify-center items-center space-y-4 z-50">
        <img src="/animation/login/login_animation.gif" alt="anim" />
        <div
          class="w-[150px] h-12 bg-red-500 py-2 px-4 rounded-full shadow-md shadow-[#161515] hover:scale-110 active:scale-100 hover:bg-red-700 flex justify-center items-center cursor-pointer transition-all duration-200 ease-in-out"
          @click="cancelLogin"
        >
          <span class="text-gray-200 text-xl font-semibold uppercase">Cancel</span>
        </div>
      </div>
    </div>
    <div class="col-start-4 col-end-22 row-start-3 row-end-11 bg-[#1a2e2c] rounded-lg p-4">
      <RemoveModal
        v-if="serverStore.isRemoveModalActive"
        @remove-handler="removeServerHandler"
        @close-window="closeRemoveModal"
      />
      <ErrorModal v-if="serverStore.errorMsgExists" :description="serverStore.error" @close-window="closeErrorDialog" />
      <form
        class="w-full h-full p-1 bg-[#305c59] col-start-1 col-span-full row-start-1 row-span-full grid grid-cols-12 grid-rows-7 gap-y-2"
        @submit.prevent="login"
      >
        <div
          class="w-full col-start-1 col-span-full row-start-1 row-span-1 h-12 bg-[#1a2e2c] p-2 grid grid-cols-12 rounded-full"
        >
          <div class="col-start-1 col-span-10 h-full">
            <select
              v-model="serverStore.selectedName"
              class="w-full h-full rounded-full px-2 text-md text-gray-800 font-semibold"
            >
              <option value="" disabled selected>Select your Server-Connection</option>
              <option v-for="connection in serverStore.connections" :key="connection.name" :value="connection.name">
                {{ connection.name }}
              </option>
            </select>
          </div>
          <div
            class="w-8 h-8 bg-slate-500 rounded-full flex justify-center items-center cursor-pointer hover:bg-slate-400 justify-self-end"
            @click.prevent="add"
          >
            <img
              class="w-6 h-6 hover:scale-110 active:scale-100 transition-all ease-in-out duration-200"
              src="/img/icon/server-management-icons/plus-icon.png"
              alt="icon"
              @mousedown.prevent
            />
          </div>
          <div
            class="w-8 h-8 bg-slate-500 rounded-full flex justify-center items-center cursor-pointer hover:bg-slate-400 justify-self-end"
            @click.prevent="removeModalHandler"
            @mouseenter="hovered = true"
            @mouseleave="hovered = false"
          >
            <img
              class="w-6 h-6 hover:scale-110 active:scale-100 transition-all ease-in-out duration-200"
              :src="getTrashImg"
              alt=""
              @mousedown.prevent
            />
          </div>
        </div>
        <div
          class="w-full col-start-1 col-span-full row-start-2 row-span-1 h-12 bg-[#1a2e2c] p-2 grid grid-cols-12 rounded-full"
          :class="{ errors: !serverStore.loginState?.hostName }"
        >
          <span class="w-full col-start-1 col-span-3 text-sm text-gray-300 font-semibold self-center pl-2">{{
            $t("formsetup.servername")
          }}</span>

          <input
            v-model="serverStore.loginState.hostName"
            class="col-start-4 col-span-full w-full h-full rounded-full px-2"
            :class="{
              notFilled: !serverStore.loginState.hostName,
              isFilled: serverStore.loginState.hostName,
            }"
            name="servername"
            type="text"
          />
        </div>
        <div
          class="w-full col-start-1 col-span-full row-start-3 row-span-1 h-12 bg-[#1a2e2c] p-2 grid grid-cols-12 rounded-full"
          :class="{ errors: !serverStore.loginState.ip }"
        >
          <span class="w-full col-start-1 col-span-3 text-sm text-gray-300 font-semibold self-center pl-2">{{
            $t("formsetup.iphost")
          }}</span>

          <div
            class="col-start-4 col-span-1 w-8 h-8 rounded-full border-4 border-slate-500 bg-gray-200 flex justify-center items-center p-1 cursor-pointer hover:border-slate-400 shadow-md shadow-[#1c2122]"
            @click="ipScanModal = true"
          >
            <img class="w-4 h-4 hover:scale-110 active:scale-100" src="/img/icon/server-management-icons/local-lan.png" alt="" />
          </div>
          <input
            v-model="serverStore.loginState.ip"
            class="col-start-5 col-span-7 w-full h-full rounded-l-full border-r border-gray-400 px-2"
            :class="{
              notFilled: !serverStore.loginState.ip,
              isFilled: serverStore.loginState.ip,
            }"
            name="host"
            type="text"
            required
          />
          <input
            v-model="serverStore.loginState.port"
            class="col-start-12 col-span-1 w-full h-full self-center flex justify-center items-center bg-gray-300 rounded-r-full cursor-pointer px-1"
            :class="{
              notFilled: !serverStore.loginState.port,
              isFilled: serverStore.loginState.port,
            }"
            type="text"
            placeholder="22"
            optional
          />
        </div>
        <div
          class="w-full col-start-1 col-span-full row-start-4 row-span-1 h-12 bg-[#1a2e2c] p-2 grid grid-cols-12 rounded-full"
          :class="{
            errors: !serverStore.loginState.username,
            isFilled: serverStore.loginState.username,
          }"
        >
          <span class="w-full col-start-1 col-span-3 text-sm text-gray-300 font-semibold self-center pl-2">{{
            $t("formsetup.username")
          }}</span>

          <input
            v-model="serverStore.loginState.username"
            class="col-start-4 col-span-full w-full h-full rounded-full px-2"
            :class="{
              notFilled: !serverStore.loginState.username,
              isFilled: serverStore.loginState.username,
            }"
            type="text"
            name="user"
            required
          />
        </div>
        <Transition name="slide-up">
          <div
            v-if="serverStore.loginState.useAuth"
            class="col-start-1 col-span-full row-start-5 row-span-2 gap-y-2 grid grid-cols-12 grid-rows-2"
          >
            <div
              class="w-full col-start-1 col-span-full row-start-1 row-span-1 h-12 bg-[#1a2e2c] p-2 grid grid-cols-12 rounded-full cursor-default"
              :class="{
                errors: serverStore.loginState.useAuth
                  ? !serverStore.loginState.keyPath
                  : !serverStore.loginState.password,
              }"
            >
              <span class="col-start-1 col-span-3 text-sm text-gray-300 font-semibold self-center pl-2">{{
                $t("formsetup.keylocation")
              }}</span>

              <input
                v-model="serverStore.loginState.keyPath"
                class="col-start-4 col-span-8 w-full h-full rounded-l-full px-2"
                type="text"
                name="keylocation"
                required
              />
              <label
                for="file_input"
                class="col-start-12 col-span-1 bg-gray-300 rounded-r-full flex justify-center items-center"
              >
                <input id="file_input" ref="fileInput" type="file" style="display: none" @change="previewFiles" />
                <img class="w-4 h-4" src="/img/icon/server-management-icons/plus.png" />
              </label>
            </div>
            <div
              class="w-full col-start-1 col-span-full row-start-2 row-span-1 h-12 bg-[#1a2e2c] p-2 grid grid-cols-12 rounded-full cursor-default"
              :class="{
                errors: serverStore.loginState.useAuth
                  ? !serverStore.loginState.keyPath
                  : !serverStore.loginState.passphrase,
              }"
            >
              <span class="col-start-1 col-span-3 text-sm text-gray-300 font-semibold self-center pl-2">{{
                $t("formsetup.KeyPassphrase")
              }}</span>
              <input
                v-model="serverStore.loginState.passphrase"
                class="col-start-4 col-span-8 w-full h-full rounded-l-full px-2"
                :class="{
                  notFilled: !serverStore.loginState.passphrase,
                  isFilled: serverStore.loginState.passphrase,
                }"
                :type="inputType"
                name="passphrase"
              />
              <div
                class="col-start-12 col-span-1 w-full h-full self-center flex justify-center items-center bg-gray-300 rounded-r-full cursor-pointer px-1"
                @click="toggleShowPassword"
              >
                <img class="w-6 h-6" src="/img/icon/server-management-icons/eye.png" alt="eyeIcon" />
              </div>
            </div>
          </div>
          <div
            v-else
            class="w-full col-start-1 col-span-full row-start-5 row-span-1 h-12 bg-[#1a2e2c] p-2 grid grid-cols-12 rounded-full cursor-default"
            :class="{
              errors: serverStore.loginState.useAuth
                ? !serverStore.loginState.useAuth
                : !serverStore.loginState.password,
            }"
          >
            <span class="w-full col-start-1 col-span-3 text-sm text-gray-300 font-semibold self-center pl-2">{{
              $t("formsetup.password")
            }}</span>

            <input
              v-model="serverStore.loginState.password"
              class="col-start-4 col-span-8 w-full h-full rounded-l-full px-2"
              :class="{
                notFilled: !serverStore.loginState.password,
                isFilled: serverStore.loginState.password,
              }"
              :type="inputType"
              name="keylocation"
              required
            />
            <div
              class="col-start-12 col-span-1 w-full h-full self-center flex justify-center items-center bg-gray-300 rounded-r-full cursor-pointer"
              @click="toggleShowPassword"
            >
              <img class="w-6 h-6" src="/img/icon/server-management-icons/eye.png" alt="eyeIcon" />
            </div>
          </div>
        </Transition>
        <div
          class="w-full h-8 col-start-5 col-span-4 justify-self-center self-end row-start-7 row-span-1 bg-[#1a2e2c] p-1 rounded-full flex justify-between items-center"
        >
          <span class="w-fit text-gray-300 font-semibold text-sm ml-1">{{ $t("formsetup.usessh") }}</span>

          <label
            for="AcceptConditions"
            class="relative h-6 w-12 cursor-pointer [-webkit-tap-highlight-color:_transparent]"
          >
            <input
              id="AcceptConditions"
              v-model="serverStore.loginState.useAuth"
              type="checkbox"
              class="peer sr-only [&:checked_+_span_svg[data-checked-icon]]:block [&:checked_+_span_svg[data-unchecked-icon]]:hidden"
              @change="changeLabel"
            />

            <span
              class="absolute inset-y-0 start-0 z-10 m-1 inline-flex h-4 w-4 items-center justify-center rounded-full bg-white text-gray-400 transition-all peer-checked:start-6 peer-checked:text-green-600"
            >
              <svg
                data-unchecked-icon
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path
                  fill-rule="evenodd"
                  d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
                  clip-rule="evenodd"
                />
              </svg>

              <svg
                data-checked-icon
                xmlns="http://www.w3.org/2000/svg"
                class="hidden h-4 w-4"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path
                  fill-rule="evenodd"
                  d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                  clip-rule="evenodd"
                />
              </svg>
            </span>

            <span class="absolute inset-0 rounded-full bg-gray-300 transition peer-checked:bg-green-500"></span>
          </label>
        </div>
        <button
          class="w-[150px] h-14 absolute bottom-4 right-4 text-center bg-[#1a2e2c] rounded-full text-gray-200 font-semibold text-xl capitalize hover:bg-[#264744] transition-all hover:scale-105 active:scale-100 hover:shadow-md active:shadow-none hover:shadow-gray-800"
        >
          {{ $t("formsetup.login") }}
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import IpScanModal from "./IpScanModal.vue";
import RemoveModal from "./RemoveModal.vue";
import ErrorModal from "./ErrorModal.vue";
import ControlService from "@/store/ControlService";
import { useServers } from "@/store/servers";

import { ref, computed, watch, onBeforeMount, watchEffect } from "vue";
import { useServerLogin } from "@/composables/useLogin";
import { useRouter } from "vue-router";

const serverStore = useServers();
const router = useRouter();
const { login, add, remove, loadStoredConnections } = useServerLogin();

//State
const abortController = ref(new AbortController());
const scannedCounter = ref(0);
const btnSearchState = ref("search");
const ipScanModal = ref(false);
const devices = ref([]);
const foundIp = ref("Searching...");
// const sshPort = ref(null);
// const link = ref("stereumLogoExtern.png");
// const stereumVersions = ref({});
const noIpFound = ref("No IP found");
const hovered = ref(false);

//Computed & Watch
const getTrashImg = computed(() => {
  if (hovered.value) {
    return "./img/icon/server-management-icons/trash-can-2.png";
  } else {
    return "./img/icon/server-management-icons/trash-can.png";
  }
});

const inputType = computed(() => {
  return serverStore.showPassword ? "text" : "password";
});

watch(devices, () => {
  console.log("devices", devices.value);
  if (devices.value.length < 1) {
    foundIp.value = noIpFound.value;
    btnSearchState.value = "search";
  } else if (devices.value.length == 1) {
    foundIp.value = devices.value[0].ip;
    btnSearchState.value = "copy";
  }
});

watchEffect(() => {
  if (serverStore.loginState.useAuth) {
    serverStore.loginState.password = "";
  }
});

watch(
  () => serverStore.selectedName,
  (newVal) => {
    const selectedConnection = serverStore.connections.find((c) => c.name === newVal);
    if (selectedConnection) {
      serverStore.loginState.hostName = selectedConnection.name;
      serverStore.loginState.ip = selectedConnection.host;
      serverStore.loginState.username = selectedConnection.user;
      serverStore.loginState.port = selectedConnection.port;
      serverStore.loginState.keyPath = selectedConnection.keylocation;
      serverStore.loginState.useAuth = selectedConnection.useAuthKey;
    }
  }
);

//Lifecycle Hooks
onBeforeMount(async () => {
  await loadStoredConnections();
});

//Methods

const changeLabel = () => {
  if (serverStore.loginState.useAuth) {
    serverStore.loginState.password = "";
  }
};

const scanFunction = () => {
  if (scannedCounter.value === 0 && btnSearchState.value === "search") {
    scannedCounter.value++;
    startScanning();
  } else if (btnSearchState.value === "search") {
    startScanning();
  } else if (btnSearchState.value === "pending") {
    return "";
  } else if (btnSearchState.value === "copy") {
    copyIp(foundIp.value);
  }
  return "";
};

const copyIp = async (arg) => {
  await navigator.clipboard.writeText(arg);
};

const startScanning = async () => {
  btnSearchState.value = "pending";
  foundIp.value = "Searching...";
  await IpScanLan1();
};

const IpScanLan1 = async () => {
  try {
    let res = await ControlService.IpScanLan();
    devices.value = res;
  } catch (error) {
    console.error("An error occurred:", error);
  }
};

const toggleShowPassword = () => {
  serverStore.showPassword = !serverStore.showPassword;
};

const previewFiles = (event) => {
  const Path = event.target.files[0].path;
  let pathString = new String(Path);
  let result = pathString.toString();
  serverStore.loginState.keyPath = result;
};

const closeErrorDialog = () => {
  serverStore.errorMsgExists = false;
};

const closeRemoveModal = () => {
  serverStore.isRemoveModalActive = false;
};

const removeModalHandler = () => {
  serverStore.isRemoveModalActive = true;
};

const removeServerHandler = async () => {
  serverStore.selectedServerToConnect = serverStore.connections.find((c) => c.name === serverStore.selectedName);
  await remove();
  serverStore.isRemoveModalActive = false;
};

const cancelLogin = () => {
  abortController.value.abort();
  serverStore.connectingAnimActive = false;
  serverStore.loginState.password = "";
  serverStore.loginState.passphrase = "";
};
</script>
<style scoped>
.slide-up-enter-active,
.slide-up-leave-active {
  transition: all 0.5s ease-out;
}

.slide-up-enter-from {
  opacity: 0;
  transform: translateX(-50px);
}

.slide-up-leave-to {
  opacity: 0;
  transform: translateY(-50px);
}
select option {
  background-color: white;
  margin-bottom: 1px;
  cursor: pointer;
  padding: 0.5em;
  border-width: 0;
}
.alert {
  width: 40%;
  height: 20%;
  background-color: #1e2429;
  border-radius: 20px;
  position: absolute;
  z-index: 7;
  top: 30%;
  border: 3px solid #bb1010;
  color: #ddbfbf;
  font-size: 135%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: 600;
}
.passwordShow {
  display: flex;
  position: absolute;
  left: 92%;
  cursor: pointer;
}
.ssvBtn {
  display: none;
}
.ssvFile-label {
  width: 57%;
  height: 80%;
  margin-right: 16px;
  border-radius: 40px;

  background: #dbdbdb;
  color: #242424 !important;
  font-size: 65%;
  font-weight: bold;
  outline-style: none;
  border: 2px solid #929292;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  flex-wrap: nowrap;
}
.server-parent {
  width: 99%;
  height: 99%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: default;
}
.server-box {
  width: 100%;
  height: 100%;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: 7% 14% 59% 20%;
}
#header {
  grid-column: 1/4;
  grid-row: 2/3;
  border: 5px solid #929292;
  margin: 0 auto;
  width: max-content;
  height: 59%;
  border-radius: 40px;
  background-color: #194747;
  opacity: 0.9;
  box-shadow: 0 1px 3px 1px #1f3737;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 0 0.7%;
}
#header span {
  width: max-content;
  max-width: auto;
  height: 89%;
  font-size: 1.4rem !important;
  font-weight: 700 !important;
  color: #cecece !important;
  border: none;
  background-color: transparent;
  box-shadow: none;
  text-transform: uppercase;
}

#dialTitle {
  animation: blink 1s 1000000 alternate;
  font-weight: bold;
}
@keyframes blink {
  from {
    background-color: red;
  }
  to {
    background-color: orange;
  }
}

.priority {
  z-index: 200;
}
.select-wrapper {
  width: 82%;
  height: 100%;
  border-radius: 40px;
  border: none;
  display: flex;
  justify-content: center;
  align-items: center;
}
.select-wrapper::after {
  width: 50%;
  height: 100%;
}

#one {
  width: 100%;
  height: 50px;
  margin: 0;
  border: none;
  border-radius: 40px;
  background-color: #1a3a33;
  justify-content: space-evenly;
  align-items: center;
  display: flex;
}
#one select {
  /* styling */
  outline-style: none;
  background-color: white;
  border-radius: 40px;
  display: inline-block;
  font-size: 14px;
  font-weight: bold;
  line-height: 1.5em;
  /* reset */
  margin: 0;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;

  color: #393939;
  font-weight: 800;
  font-size: 1rem;
}

#two {
  width: 70%;
  padding: 1rem;
  border-radius: 40px;
  float: left;
}
.three {
  width: 30px;
  height: 30px;
  margin-right: 5px;
  border-radius: 50%;
  outline-style: none;
  box-shadow: 0 0 3px 1px rgb(149, 149, 149);
}
.three:active {
  box-shadow: none;
}
.plus:hover {
  background-color: green;
}
.trash:hover {
  background-color: rgb(230, 84, 81);
}
.three img {
  width: 30px;
  height: 30px;
  cursor: pointer;
}
.three:hover img {
  transform: scale(1.1);
  transition-duration: 100ms;
}
.three:active img {
  transform: scale(1);
  transition-duration: 100ms;
}
.server-group {
  height: 20%;
  width: 100%;
  display: flex;
  border: none;
  align-items: center;
  justify-content: space-between;
  font-weight: bold;
  box-sizing: border-box;
}
.labelBox {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  height: 100%;
  width: 30%;
  font-size: 100%;
  font-weight: 700;
  color: #dfdfdf !important;
}
.server-group_input {
  width: 70%;
  height: 100%;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}

.server-group_input input {
  width: 86%;
  height: 80%;
  background-color: #eaeaea;
  border: 2px solid #979797;
  border-radius: 40px;
  padding-left: 10px;
  font-weight: 600 !important;
  font-size: 0.9rem !important;
  outline-style: none !important;
  color: #242424;
}
.server-group_input input:hover {
  border: 2px solid rgb(54, 54, 54);
}
#iporhostname {
  width: 60%;
  border-radius: 30px 0 0 30px;
}
.ip-scaner {
  width: 7.5%;
  height: 80%;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #eaeaea;
  margin-right: 2%;
  border: 2px solid #979797;
  border-radius: 50%;
  cursor: pointer;
}
.ip-scaner img {
  width: 80%;
  opacity: 90%;
}
.ip-scaner:active {
  transform: scale(0.95);
}
.ipPort {
  width: 16% !important;
  height: 80% !important;
  border-radius: 0 30px 30px 0 !important;
}
#keyLocation {
  width: 65%;
  border: 5px solid #929292;
  border-radius: 14px;
  background-color: #234141;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 14%;
  box-shadow: 0 1px 3px 1px #182f2f;
  z-index: 95;
  position: relative;
}
.keyLocation_title {
  clear: both;
  font-size: 90%;
  font-weight: 700;
  color: #cecece !important;
  margin-left: 24px;
}
.chooseFile {
  cursor: pointer;
  left: 41%;
  top: 17%;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 70%;
  width: 5%;
  position: absolute;
  box-sizing: border-box;
  border-right: 3px solid #929292;
}
.chooseFile img {
  width: 50%;
  opacity: 80%;
}
.chooseFile:active,
.chooseFile:focus {
  border: none;
  transform: scale(0.9);
}
#keyLocation input {
  width: 57%;
  height: 80%;
  margin-right: 16px;
  border-radius: 40px;
  padding-left: 10px;
  background-color: #dbdbdb;
  font-size: 80%;
  font-weight: 600;
  outline-style: none;
  border: 2px solid #929292;
  display: flex;
  justify-content: flex-start;
  align-items: center;
}
.locationPicker {
  display: flex;
  height: 80%;
  width: 60%;
}
#keyInput {
  height: 100% !important;
  width: 96% !important;
  margin-left: 1%;
  padding-left: 9% !important;
}

#login {
  width: 120px;
  min-width: 120px;
  height: 48px;
  min-height: 45px;
  outline-style: none;
  padding: 5px;
  border: 5px solid #929292;
  border-radius: 35px;
  cursor: pointer;
  position: absolute;
  right: 8%;
  bottom: 7%;
  background-color: #264c4c;
  box-shadow: 0 1px 3px 1px rgb(23, 38, 32);
  font-size: 1.4rem;
  font-weight: 800;
  color: #cecece;
  display: flex;
  justify-content: center;
  align-items: center;
}
#login:hover {
  box-shadow: none;
  border: 5px solid #464646;
  background-color: #1b3737;
}
#login:active {
  box-shadow: inset 0 1px 5px 2px rgb(23, 38, 32);
}
input {
  cursor: pointer;
}

.ssh {
  width: 150px;
  min-width: 100px;
  height: 34px;
  background-color: #234141;
  border: 3px solid #929292;
  border-radius: 40px;
  color: rgb(235, 235, 235);
  position: absolute;
  left: 17.5%;
  bottom: 15%;
  box-shadow: 0 1px 3px 1px rgb(23, 38, 32);
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 1px;
}
#lbl {
  width: max-content;
  text-align: center;
  clear: both;
  font-weight: bold;
  font-size: 70%;
  display: flex;
}
.switch {
  position: relative;
  display: inline-block;
  width: 30%;
  height: 89%;
  margin-left: 2px;
  margin-top: 3px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  height: 22px;
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgb(216, 216, 216);
  -webkit-transition: 0.4s;
  transition: 0.4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  border: 1px solid #1b9dcc;
  left: 6%;
  bottom: 9%;
  background-color: #25acde;
  -webkit-transition: 0.4s;
  transition: 0.4s;
}

input:checked + .slider {
  background-color: #e6e6e6;
}

input:checked + .slider:before {
  -webkit-transform: translateX(20px);
  -ms-transform: translateX(10px);
  transform: translateX(19px);
  background-color: #51a76e;
  border: 1px solid #2d944f;
}

/* Rounded sliders */
.slider.round {
  border-radius: 24px;
}

.slider.round:before {
  border-radius: 50%;
}

.error-box {
  width: 100%;
  height: 100%;
  background-color: rgb(8, 8, 8);
  opacity: 0.7;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 96;
}
.error-modal {
  width: 35%;
  height: 40%;
  background-color: rgb(232, 232, 232);
  box-shadow: 0px 1px 3px 1px rgb(19, 19, 19);
  border-radius: 10px;
  position: absolute;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  z-index: 101;
}
.error-modal .title-box {
  grid-column: 1/2;
  grid-row: 1/4;
  width: 100%;
  height: 100%;
  border-radius: 9px 9px 0 0;
  display: flex;
  justify-content: center;
  align-items: center;
}
.title-box img {
  width: 90%;
  height: 75%;
  margin-left: 10px;
}
.error-modal .description {
  grid-column: 2/4;
  grid-row: 1/3;
  width: 100%;
  height: 100%;
  margin-top: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.error-modal .description span {
  width: 100%;
  height: 50%;
  margin-right: 10px;
  color: #e81f05;
  font-size: 1.2rem;
  font-weight: 800;
  text-transform: uppercase;
  text-align: center;
}

.error-modal .btn-box {
  grid-column: 2/4;
  grid-row: 3;
  width: 100%;
  height: 50%;

  display: flex;
  justify-content: center;
  align-items: center;
}
.btn-box button {
  width: 60%;
  height: 80%;
  margin-right: 10px;
  outline-style: none;
  background-color: #e81f05;
  border: 1px solid rgb(178, 178, 178);
  border-radius: 10px;
  color: #eae9e9;
  font-size: 1rem;
  font-weight: 800;
  box-shadow: 0px 1px 3px 1px #811515;
  transition-duration: 100ms;
}
.btn-box button:hover {
  transform: scale(1.1);
  border: 1px solid rgb(59, 11, 11);
}
.btn-box button:active {
  transform: scale(1);
  box-shadow: none;
}
.anim {
  width: 100%;
  height: 100%;
  background-color: rgba(8, 8, 8, 0.85);
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 99;
}
.anim img {
  width: 35%;
  height: 45%;
}
.cancl-btn {
  width: 20%;
  height: 10%;
  position: absolute;
  top: 80%;
  right: 40%;
  background-color: #eb5353;
  border-radius: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  font-size: 120%;
  font-weight: 800;
  color: #eae9e9;
  letter-spacing: 1px;
  text-transform: uppercase;
  cursor: pointer;
}
.cancl-btn:active {
  transform: scale(0.95);
}
.error {
  color: #e43e3e;
  border-color: #e43e3e !important;
  box-shadow: none;
}
.notFilled {
  color: rgb(51, 51, 51);
  border-color: rgb(226, 182, 86) !important;
  box-shadow: none;
}
</style>
