<script>
  import { onMount } from "svelte";
  import QrScanner from "qr-scanner";
  import pb from "./lib/db";
  import { AppBar } from '@skeletonlabs/skeleton';
  import { Toast, toastStore } from "@skeletonlabs/skeleton";

  // Your selected Skeleton theme:
  import "@skeletonlabs/skeleton/themes/theme-skeleton.css";
  // This contains the bulk of Skeletons required styles:
  import "@skeletonlabs/skeleton/styles/all.css";

  let stream;
  let pos;
  let qrScanner;

  onMount(() => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        pos = {
          lat: position.coords.latitude,
          lng: position.coords.longitude,
        };
      });
    }
    qrScanner = new QrScanner(
      stream,
      async (result) => {
        const scanned_data = JSON.parse(result.data);
        qrScanner.stop();
        const t = {
          message: `escaneo válido: ${scanned_data.data}`,
        };
        toastStore.trigger(t);
        const data = {
          package: scanned_data.id,
          lat: pos.lat,
          lng: pos.lng,
        };
        const response = await pb.collection("locations").create(data);
        console.log(response);
      },
      {
        highlightScanRegion: true,
      }
    );
  });
</script>

<div class="flex flex-col min-h-screen bg-secondary-300">
  <Toast position="t"/>
  <div class="flex-grow item">
    <AppBar background="bg-primary-500" gridColumns="grid-cols-1" slotDefault="place-self-center">
      <h3>scanner</h3>
    </AppBar>
    <!-- svelte-ignore a11y-media-has-caption -->
    <video bind:this={stream} />
    <div class="flex justify-around mt-3 self-end">
      <button
        type="button"
        class="btn variant-filled"
        on:click={qrScanner.start()}>scan</button
      >
      <button
        type="button"
        class="btn variant-filled"
        on:click={qrScanner.stop()}>stop</button
      >
    </div>
  </div>
</div>
