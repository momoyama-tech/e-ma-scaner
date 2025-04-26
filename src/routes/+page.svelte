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
        const cropX = 300;   // ← 必要なら調整して
        const cropY = 80;   // ← 必要なら調整して
        const cropWidth = 1000;  // ← 必要なら調整して
        const cropHeight = 950;  // ← 必要なら調整して

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
  
<h1>カメラ切り替え付き版</h1>
  
<p>カメラ選択:</p>
<select on:change={handleDeviceChange} bind:value={selectedDeviceId}>
    {#each devices as device}
      <option value={device.deviceId}>{device.label || `カメラ ${device.deviceId}`}</option>
    {/each}
</select>
  
<video bind:this={videoElement} autoplay playsinline width="640" height="480">
    <track kind="captions" src="" srclang="en" label="No captions available" />
</video>
  
{#if previewUrl}
    <img src={previewUrl} alt="撮影した画像" width="400" />
{/if}
  
<canvas bind:this={canvasElement} width="1920" height="1080" style="display: none;"></canvas>
  
<p>Enterキーを押すと現在選択中のカメラから撮影（ピンク枠だけ切り取り）→ アップロードします</p>
