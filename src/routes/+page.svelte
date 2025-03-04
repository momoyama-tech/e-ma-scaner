<script>
    let file;
    let previewUrl = '';
    const uploadUrl = import.meta.env.VITE_BACKEND_URL;
  
    function handleFileChange(event) {
      file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => previewUrl = e.target.result;
        reader.readAsDataURL(file);
      }
    }
  
    async function uploadFile() {
      if (!file) {
        alert('ファイルを選択してください');
        return;
      }
  
      const formData = new FormData();
      formData.append('file', file);
  
      try {
        const res = await fetch(`${uploadUrl}/illusts`, { // ここを実際のAPIに変更
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
  </script>
  
  <h1>Welcome to SvelteKit</h1>
  <p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p>
  
  <input type="file" accept="image/*" on:change={handleFileChange} />
  {#if previewUrl}
    <img src={previewUrl} alt="Preview" width="200" />
  {/if}
  <button on:click={uploadFile}>アップロード</button>
  