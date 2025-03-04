<script>
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
  
      // 画像をデータURLとして取得
      previewUrl = canvasElement.toDataURL('image/png');
  
      // データURLをBlobに変換
      canvasElement.toBlob((blob) => {
        file = new File([blob], "captured_image.png", { type: "image/png" });
  
        // 画像を撮影したらすぐにアップロードする
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
  
    async function handleEnterKey(event) {
      if (event.key === "Enter") {
        event.preventDefault(); // フォームなどのデフォルト動作を防ぐ
        await startCamera(); // カメラ起動
        setTimeout(captureImage, 2000); // 2秒後に撮影（カメラが起動するのを待つ）
      }
    }
  
    // キーボードイベントを登録
    window.addEventListener("keydown", handleEnterKey);
  </script>
  
  <h1>Welcome to SvelteKit</h1>
  <p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p>
  
  <!-- カメラ映像を表示する -->
  <video bind:this={videoElement} autoplay playsinline></video>
  
  <!-- 撮影した画像を表示 -->
  {#if previewUrl}
    <img src={previewUrl} alt="Captured Image" width="200" />
  {/if}
  
  <!-- 画像を保存するためのcanvas -->
  <canvas bind:this={canvasElement} width="640" height="480" style="display: none;"></canvas>
  
  <p>Enterキーを押すとカメラ起動 → 撮影 → アップロードが実行されます</p>
  