<script>
  import { onMount } from "svelte"
  import QrScanner from "qr-scanner"
  import PocketBase from "pocketbase"

  const pb = new PocketBase(import.meta.env.VITE_DB_URL)

  let stream
  let pos
  let qrScanner

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
        const scanned_data = JSON.parse(result.data)
        
          qrScanner.stop()
          alert('scanned correctly')
          const data = {
            "package": scanned_data.id,
            "lat": pos.lat,
            "lng": pos.lng
          };
          const response = await pb.collection('locations').create(data);
          console.log(response)
      },
      {
        highlightScanRegion: true,
      }
    );
  });
</script>

<!-- svelte-ignore a11y-media-has-caption -->
<video bind:this={stream} />
<button on:click={qrScanner.start()}>scan</button>
<button on:click={qrScanner.stop()}>stop</button>

