<script>
    import { onMount } from "svelte";
  
    let videoElement;
    let canvasElement;
    let file;
    let previewUrl = '';
    const uploadUrl = import.meta.env.VITE_BACKEND_URL;
  
    async function startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        videoElement.srcObject = stream;
      } catch (error) {
        console.error('カメラを起動できませんでした:', error);
        alert('カメラを使用できません。');
      }
    }
  
    function captureImage() {
      const context = canvasElement.getContext('2d');
      context.drawImage(videoElement, 0, 0, canvasElement.width, canvasElement.height);
  
      previewUrl = canvasElement.toDataURL('image/png');
  
      canvasElement.toBlob((blob) => {
        file = new File([blob], "captured_image.png", { type: "image/png" });
        uploadFile();
      }, "image/png");
    }
  
    async function uploadFile() {
      if (!file) {
        alert('画像を撮影してください');
        return;
      }
  
      const formData = new FormData();
      formData.append('file', file);
  
      try {
        const res = await fetch(`${uploadUrl}/illusts`, {
          method: 'POST',
          body: formData
        });
  
        if (res.ok) {
          alert('アップロード成功');
        } else {
          alert('アップロード失敗');
        }
      } catch (error) {
        console.error(error);
        alert('アップロード時にエラーが発生しました');
      }
    }
  
    function handleEnterKey(event) {
      if (event.key === "Enter") {
        event.preventDefault();
        startCamera().then(() => {
          setTimeout(captureImage, 2000);
        });
      }
    }
  
    onMount(() => {
      window.addEventListener("keydown", handleEnterKey);
    });
  </script>
  
  <h1>Welcome to SvelteKit</h1>
  <p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p>
  
  <video bind:this={videoElement} autoplay playsinline>
    <track kind="captions" src="" srclang="en" label="No captions available" />
  </video>
  
  {#if previewUrl}
    <img src={previewUrl} alt="撮影した画像" width="200" />
  {/if}
  
  <canvas bind:this={canvasElement} width="640" height="480" style="display: none;"></canvas>
  
  <p>Enterキーを押すとカメラ起動 → 撮影 → アップロードが実行されます</p>
  