<script>
  import promotionImage2 from "../assets/promotion/promotion2.jpg";
  import promotionImage3 from "../assets/promotion/promotion3.jpg";
  import promotionVideo from "../assets/promotion/video.mp4";
  import arrowDown from "../assets/icons/arrow-down-white.svg";
  import warningIcon from "../assets/icons/warning.svg";
  import Button from "../lib/components/Button.svelte";
  import Dialog from "../lib/components/Dialog.svelte";
  import { createDialog } from "svelte-headlessui";
  import { Link } from "svelte-routing";
  import { Splide, SplideSlide } from "@splidejs/svelte-splide";
  import "@splidejs/svelte-splide/css";
  import logStore from "../lib/stores/logStore";
  import { invoke } from "@tauri-apps/api";
  import globalStore from "../lib/stores/globalStore";
  import { appWindow } from "@tauri-apps/api/window";
  let consoleOpen = true;
  let shareLog = false;
  let logLink = "";
  let logs = ""
  let dialog = createDialog({ label: "failed" });
  let dialogCheckLogs = createDialog({ label: "check-logs" });
  // listen to install fail event
  function installFail() {
    if ($logStore.installFailed) {
      console.log("Event fail received");
      dialog.open();
      console.log("Dialog should be open now");
    }
  }

  // save config. This triggers the backend install
  async function saveConf() {
    await invoke("install", { data: JSON.stringify($globalStore) });
  }

  async function shareLogs() {
    shareLog = true;
    logLink = await invoke("share_logs");
    console.log(logLink);
  }
  saveConf();
  $: $logStore, installFail();
</script>

<Dialog {dialog}>
  <div
    class="flex flex-col justify-center items-center text-center p-6 space-y-4"
  >
    <img src={warningIcon} alt="" />
    <div class="text-4xl font-medium">Installation Failed</div>
    <!--The link to termbin.com-->
    {#if shareLog}
      <div>{logLink}</div>
    {/if}
    <!-- svelte-ignore a11y-invalid-attribute -->
    <button
      class="text-xs hover:text-cyan-400"
      on:click={async () => {
        logs = await invoke("get_all_logs")
        dialogCheckLogs.open()

        }}>Check logs ?</button
    >
    <Button fullWidth variant="bordered" on:click={shareLogs}
      >Do you want to share the logs ?</Button
    >
    <Button fullWidth on:click={async () => await appWindow.close()}
      >Close</Button
    >
  </div>
</Dialog>

<Dialog dialog={dialogCheckLogs}>
  <div class="grow overflow-scroll bg-gray-800 rounded-xl w-full px-3 py-2" style="height:400px">
    <pre class="w-full whitespace-pre-line">
        {logs}      
    </pre>
  </div>
  <Button fullWidth on:click={() => {dialogCheckLogs.close(); dialog.open()}}>Close</Button>
</Dialog>

<main
  class="h-full p-4 space-y-4 absolute top-0 left-0 right-0 overflow-scroll bg-gradient-to-tr from-blue-700 to-indigo-700"
>
  <div class="text-center">
    <h2 class="text-center py-2">Installing</h2>
    <div>Sit back and enjoy while we install AthenaOS for you</div>
  </div>
  <div class="h-[calc(100%-96px)] space-y-4 flex flex-col">
    <div class="w-full h-fit relative">
      <div
        class="px-4 py-1 bg-white text-black uppercase text-sm font-medium rounded-full absolute top-2 right-2 z-10"
      >
        Promotions
      </div>
      <Splide
        class="w-full rounded-xl overflow-hidden"
        aria-label="My Favorite Images"
        options={{ type: "loop" }}
      >
        <SplideSlide>
          <div class="w-full h-full relative">
            <video
              class={`object-cover w-full ${
                consoleOpen ? "aspect-[38/9]" : "aspect-[20/9]"
              }`}
              autoplay
            >
              <source src={promotionVideo} type="video/mp4" />
              <track kind="captions" />
            </video>
            <div
              class="absolute top-0 bottom-0 left-0 right-0 h-full w-full flex p-4"
            >
              <Button>Visit Sponsor</Button>
            </div>
          </div>
        </SplideSlide>
        <SplideSlide>
          <div class="w-full h-full relative">
            <img
              class={`object-cover w-full ${
                consoleOpen ? "aspect-[38/9]" : "aspect-[20/9]"
              }`}
              src={promotionImage3}
              alt="Image1"
            />
            <div
              class="absolute top-0 bottom-0 left-0 right-0 h-full w-full flex p-4"
            >
              <Button>Visit Sponsor</Button>
            </div>
          </div>
        </SplideSlide>
        <SplideSlide>
          <div class="w-full h-full relative">
            <img
              class={`object-cover w-full ${
                consoleOpen ? "aspect-[38/9]" : "aspect-[20/9]"
              }`}
              src={promotionImage2}
              alt="Image1"
            />
            <div
              class="absolute top-0 bottom-0 left-0 right-0 h-full w-full flex p-4"
            >
              <Button>Visit Sponsor</Button>
            </div>
          </div>
        </SplideSlide>
      </Splide>
    </div>
    <div class="flex space-x-4">
      <button
        on:click={() => (consoleOpen = !consoleOpen)}
        class="flex items-center justify-center bg-gray-800 h-10 px-4 rounded-xl"
        ><img src={arrowDown} alt="" /></button
      >
      <div
        class="h-10 bg-gray-800 rounded-xl w-full flex items-center px-4 py-2"
      >
        <div class="relative h-2 w-full rounded-full overflow-hidden">
          <div
            class="absolute top-0 bottom-0 left-0 my-auto w-full bg-gray-700"
          />
          <div
            style="width: {$logStore.progress}%;"
            class="absolute top-0 bottom-0 left-0 my-auto bg-primary-500"
          />
        </div>
      </div>
    </div>
    {#if consoleOpen}
      <div class="grow overflow-scroll bg-gray-800 rounded-xl w-full px-3 py-2">
        <pre class="w-full whitespace-pre-line">
        {#each $logStore.logs as log}
            {log}
          <br />
          {/each}
        </pre>
      </div>
    {/if}
  </div>
</main>
