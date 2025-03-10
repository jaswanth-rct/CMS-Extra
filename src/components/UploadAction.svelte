<script lang="ts">
    import { Button, Modal, Fileupload, Select, Radio } from "flowbite-svelte";
    import Upload from "phosphor-svelte/lib/Upload";
    import { fetchAuthorized } from "@/lib/api";
    import { invalidate } from '$app/navigation';
    import iso6391 from 'iso-639-1';

    export let assetId: string;

    let isOpen = false;
    let files: FileList;
    let locale = "en";
    let fileType = "subtitle"; // Default to subtitle upload

    const uploadFile = async () => {
        if (files.length != 1 || !locale) {
            return;
        }

        const formData = new FormData();
        formData.append("file", files[0]);
        formData.append("locale", locale);
        formData.append("type", fileType);

        const endpoint = fileType === "subtitle" ? `/api/assets/${ assetId }/tracks` : `/api/assets/${ assetId }/audio`;
        
        await fetchAuthorized(endpoint, {
            method: 'POST',
            body: formData
        });
        
        await invalidate(`/api/assets/${ assetId }`);
    };

    const locales = iso6391.getAllNames().map(name => {
        return {
            name, value: iso6391.getCode(name)
        };
    });
</script>

<Button class="with-icon" on:click={() => { isOpen = true }}>
    Upload
    <Upload />
</Button>
<Modal title="Upload" bind:open={isOpen} autoclose>
    <label>File Type</label>
    <div class="flex space-x-4 mb-4">
        <Radio id="subtitle" name="fileType" value="subtitle" bind:group={fileType} checked>
            Subtitle (.vtt, .srt)
        </Radio>
        <Radio id="audio" name="fileType" value="audio" bind:group={fileType}>
            Audio (.mp3, .wav, .aac)
        </Radio>
    </div>

    <label for="file-upload">File</label>
    <Fileupload
        id="file-upload"
        accept={fileType === "subtitle" ? ".vtt,.srt" : ".mp3,.wav,.aac"}
        bind:files
        required
    />
    <p>Allowed file types: {fileType === "subtitle" ? ".vtt, .srt" : ".mp3, .wav, .aac"}</p>

    <label for="file-locale">Locale</label>
    <Select id="file-locale" bind:value={locale} required>
        {#each locales as locale}
            <option value={locale.value}>{locale.name}</option>
        {/each}
    </Select>

    <svelte:fragment slot="footer">
        <Button on:click={uploadFile}>Upload</Button>
    </svelte:fragment>
</Modal>
