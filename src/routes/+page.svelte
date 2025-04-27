<script>
    import { onMount } from "svelte";
  
    let videoElement;
    let canvasElement;
    let file;
    let previewUrl = '';
    const uploadUrl = import.meta.env.VITE_BACKEND_URL;
  
    let devices = [];
    let selectedDeviceId = '';
  
    async function getCameras() {
        try {
            const allDevices = await navigator.mediaDevices.enumerateDevices();
            devices = allDevices.filter(device => device.kind === 'videoinput');
            if (devices.length > 0) {
                selectedDeviceId = devices[0].deviceId;
                await startCamera(selectedDeviceId);
            }
        } catch (error) {
            console.error('カメラ一覧の取得に失敗しました:', error);
        }
    }
  
    let stream = null;
    
    async function startCamera(deviceId) {
        try {
            if (stream) {
                // すでに起動している場合は停止する
                stream.getTracks().forEach(track => track.stop());
            }
  
            stream = await navigator.mediaDevices.getUserMedia({
                video: { deviceId: { exact: deviceId } }
            });
            videoElement.srcObject = stream;
        } catch (error) {
            console.error('カメラを起動できませんでした:', error);
            alert('カメラを使用できません。');
        }
    }
  
    function captureImage() {
        const context = canvasElement.getContext('2d');
        context.drawImage(videoElement, 0, 0, canvasElement.width, canvasElement.height);
  
        // 🔥ここから追記：ピンク枠だけ切り取る処理
        const cropX = 390;   // ← 必要なら調整して
        const cropY = 75;   // ← 必要なら調整して
        const cropWidth = 900;  // ← 必要なら調整して
        const cropHeight = 1000;  // ← 必要なら調整して

        const croppedCanvas = document.createElement('canvas');
        croppedCanvas.width = cropWidth;
        croppedCanvas.height = cropHeight;
        const croppedContext = croppedCanvas.getContext('2d');

        croppedContext.drawImage(
            canvasElement,
            cropX, cropY, cropWidth, cropHeight,
            0, 0, cropWidth, cropHeight
        );

        previewUrl = croppedCanvas.toDataURL('image/png');

        croppedCanvas.toBlob((blob) => {
            file = new File([blob], "captured_image.png", { type: "image/png" });
            uploadFile();
        }, "image/png");
        // 🔥ここまで追記
    }
  
    async function uploadFile() {
        if (!file) {
            alert('画像を撮影してください');
            return;
        }
  
        const formData = new FormData();
        formData.append('image', file);
  
        try {
            const res = await fetch(`${uploadUrl}/emas`, {
                method: 'POST',
                body: formData
            });
  
            const responseJson = await res.json();
  
            if (res.ok) {
                alert('アップロード成功');
            } else {
                alert(`アップロード失敗: ${responseJson.error || '不明なエラー'}`);
            }
        } catch (error) {
            console.error(error);
            alert('アップロード時にエラーが発生しました');
        }
    }
  
    function handleEnterKey(event) {
        if (event.key === "Enter") {
            event.preventDefault();
            captureImage();
        }
    }
  
    function handleDeviceChange(event) {
        selectedDeviceId = event.target.value;
        startCamera(selectedDeviceId);
    }
  
    onMount(() => {
        getCameras();
        window.addEventListener("keydown", handleEnterKey);
    });
</script>

<style>
  body {
    background-color: #FFF4E6; /* 優しい肌色の背景 */
    color: #333;
    font-family: 'Comic Sans MS', cursive, sans-serif; /* 可愛らしいフォント */
    margin: 0;
    padding: 0;
  }

  .header {
    text-align: center;
    padding: 20px;
    background-color: #FFB6C1; /* ピンク色の背景 */
    color: white;
    font-size: 36px;
    font-weight: bold;
    border-bottom: 4px solid #FF69B4; /* 濃いピンクのボーダー */
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }

  .description {
    text-align: center;
    margin: 20px 0;
    font-size: 18px;
    color: #555;
  }

  .button-container {
    text-align: center;
    margin-bottom: 20px;
  }

  .button-container button {
    background-color: #FF69B4; /* 濃いピンクのボタン */
    color: white;
    border: none;
    border-radius: 20px;
    padding: 10px 30px;
    font-size: 18px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .button-container button:hover {
    background-color: #FF1493; /* ダークピンク */
  }

  .container {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    justify-content: space-between; /* 要素間のスペースを均等に */
    gap: 20px;
    padding: 20px;
    text-align: center;
    max-width: 1200px; /* 最大幅を設定 */
    margin: 0 auto; /* 中央揃え */
  }

  .section {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    border: 2px solid #FF69B4; /* 濃いピンクの枠線 */
    border-radius: 20px;
    padding: 20px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    background-color: #fff;
  }

  .section h2 {
    margin-bottom: 10px;
    font-size: 20px;
    color: #FF69B4;
  }

  video, .image-placeholder {
    border: 2px solid #FF69B4; /* 濃いピンクの枠線 */
    border-radius: 20px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    width: 100%; /* 横幅を親要素に合わせる */
    max-width: 600px; /* 最大幅を設定 */
    height: auto;
    aspect-ratio: 4 / 3; /* アスペクト比を固定 */
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #f9f9f9; /* 背景色を追加 */
  }

  .image-placeholder img {
    width: 100%;
    height: auto;
    border-radius: 20px;
  }

  canvas {
    display: none;
  }

  @media (max-width: 768px) {
    .container {
      flex-direction: column;
    }

    .section {
      width: 100%;
    }

    video, .image-placeholder {
      max-width: 100%; /* モバイルでは幅を100%に */
    }
  }
</style>

<div class="header">E-ma お絵描き撮影</div>
<p>カメラ選択:</p>
<select on:change={handleDeviceChange} bind:value={selectedDeviceId}>
    {#each devices as device}
      <option value={device.deviceId}>{device.label || `カメラ ${device.deviceId}`}</option>
    {/each}
</select>
<div class="description">
  <p>撮影ボタンを押して、カメラでお絵描きを撮影しましょう！</p>
</div>

<div class="container">
  <!-- カメラセクション -->
  <div class="section">
    <h2>カメラ</h2>
    <p>カメラを使用して画像を撮影します。</p>
    <video bind:this={videoElement} autoplay playsinline width="640" height="480">
        <track kind="captions" src="" srclang="en" label="No captions available" />
    </video>
  </div>

  <!-- 撮影した写真セクション -->
  <div class="section">
    <h2>撮影した写真</h2>
    <p>撮影した画像がここに表示されます。</p>
    <div class="image-placeholder">
        {#if previewUrl}
            <img src={previewUrl} alt="撮影した画像" width="400" />
        {/if}
    </div>
  </div>
</div>

<canvas bind:this={canvasElement} width="1920" height="1080" style="display: none;"></canvas>
