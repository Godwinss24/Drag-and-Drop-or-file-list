<script setup lang="ts">
// Define the structure of each file object with URL, name, size, and type
interface FileData {
  url: string
  name: string
  size: number
  type: string
}

const uploadedFiles = ref<FileList | null>(null)
const fileUrl = ref<FileData[]>([])

watch(uploadedFiles, (newValue) => {
  if (newValue && newValue.length > 0) {
    // Create an array to store the new URLs
    const newUrls: FileData[] = []

    // Loop through the newly selected files
    for (let i = 0; i < newValue.length; i++) {
      const file = newValue[i]
      // Check if the file is already in fileUrl to prevent duplicates
      if (
        !fileUrl.value.some((existingFile) => existingFile.name === file.name)
      ) {
        newUrls.push({
          url: URL.createObjectURL(file),
          name: file.name,
          size: file.size,
          type: file.type,
        })
      }
    }

    // Append new URLs to the existing ones in fileUrl
    fileUrl.value = [...fileUrl.value, ...newUrls] // Merge with existing URLs
  } else {
    fileUrl.value = [] // Reset if no files are selected
  }
  console.log('file urls', fileUrl.value)
  console.log('uploaded files', uploadedFiles.value)
})

function removeFileFromFileList(fileName: string) {
  if (uploadedFiles.value) {
    // Convert FileList to an array
    const filesArray = Array.from(uploadedFiles.value)

    // Filter out the file with the specified name
    const filteredFiles = filesArray.filter((file) => file.name !== fileName)

    // Create a new DataTransfer object
    const dataTransfer = new DataTransfer()

    // Add the remaining files to the DataTransfer object
    filteredFiles.forEach((file) => dataTransfer.items.add(file))

    // Update the uploadedFiles to the new FileList
    uploadedFiles.value = dataTransfer.files
  }

  // Remove the corresponding URL from fileUrl
  fileUrl.value = fileUrl.value.filter((file) => file.name !== fileName)
}
</script>

<template>
  <div class="max-w-xl">
    <BaseInputFileHeadless
      v-slot="{ open, remove, preview, drop, files }"
      v-model="uploadedFiles"
      multiple
      accept="image/*"
    >
      <!-- Controls -->
      <div class="mb-4 flex items-center gap-2">
        <button
          type="button"
          class="nui-focus border-muted-200 hover:border-primary-500 text-muted-700 dark:text-muted-200 hover:text-primary-600 dark:border-muted-700 dark:bg-muted-800 dark:hover:border-primary-500 dark:hover:text-primary-600 relative flex size-8 cursor-pointer items-center justify-center rounded-full border bg-white transition-colors duration-300"
          tooltip="Select files"
          @click="open"
        >
          <Icon name="lucide:plus" class="size-4" />

          <span class="sr-only">Select files</span>
        </button>

        <button
          type="button"
          class="nui-focus border-muted-200 hover:border-primary-500 text-muted-700 dark:text-muted-200 hover:text-primary-600 dark:border-muted-700 dark:bg-muted-800 dark:hover:border-primary-500 dark:hover:text-primary-600 relative flex size-8 cursor-pointer items-center justify-center rounded-full border bg-white transition-colors duration-300"
          tooltip="Start Upload"
        >
          <Icon name="lucide:arrow-up" class="size-4" />

          <span class="sr-only">Start Upload</span>
        </button>
      </div>

      <div
        role="button"
        tabindex="-1"
        @dragenter.stop.prevent
        @dragover.stop.prevent
        @drop="drop"
      >
        <div
          v-if="!fileUrl.length"
          class="nui-focus border-muted-300 dark:border-muted-700 hover:border-muted-400 focus:border-muted-400 dark:hover:border-muted-600 dark:focus:border-muted-700 group cursor-pointer rounded-lg border-[3px] border-dashed p-8 transition-colors duration-300"
          tabindex="0"
          role="button"
          @click="open"
          @keydown.enter.prevent="open"
        >
          <div class="p-5 text-center">
            <Icon
              name="mdi-light:cloud-upload"
              class="text-muted-400 group-hover:text-primary-500 group-focus:text-primary-500 mb-2 size-10 transition-colors duration-300"
            />

            <h4 class="text-muted-400 font-sans text-sm">
              Drop files to upload
            </h4>

            <div>
              <span
                class="text-muted-400 font-sans text-[0.7rem] font-semibold uppercase"
              >
                Or
              </span>
            </div>

            <label
              for="file"
              class="text-muted-400 group-hover:text-primary-500 group-focus:text-primary-500 cursor-pointer font-sans text-sm underline underline-offset-4 transition-colors duration-300"
            >
              Select files
            </label>
          </div>
        </div>

        <ul v-else class="mt-6 space-y-2">
          <li v-for="(file, index) in fileUrl" :key="index">
            <div
              class="border-muted-200 dark:border-muted-700 dark:bg-muted-800 relative flex items-center justify-end gap-2 rounded-xl border bg-white p-3"
            >
              <div class="flex items-center gap-2">
                <div class="shrink-0">
                  <img
                    v-if="fileUrl"
                    class="size-14 rounded-xl object-cover object-center w-10 h-10"
                    :src="file.url"
                    alt="Image preview"
                  />

                  <!-- <img
                    v-else
                    class="size-14 rounded-xl object-cover object-center"
                    src="/img/avatars/placeholder-file.png"
                    alt="Image preview"
                  /> -->
                </div>

                <div class="font-sans">
                  <span
                    class="text-muted-800 dark:text-muted-100 line-clamp-1 block text-sm"
                  >
                    {{ file.name }}
                  </span>

                  <span class="text-muted-400 block text-xs">
                    {{ formatFileSize(file.size) }}
                  </span>
                </div>
              </div>

              <div
                class="ms-auto w-32 px-4 transition-opacity duration-300"
                :class="'opacity-100'"
              >
                <BaseProgress :value="0" size="xs" :color="'success'" />
              </div>

              <div class="flex gap-2">
                <button
                  class="border-muted-200 hover:border-primary-500 text-muted-700 dark:text-muted-200 hover:text-primary-600 dark:border-muted-700 dark:bg-muted-900 dark:hover:border-primary-500 dark:hover:text-primary-600 relative flex size-8 cursor-pointer items-center justify-center rounded-full border bg-white transition-colors duration-300 disabled:cursor-not-allowed disabled:opacity-60"
                  disabled
                  type="button"
                  tooltip="Cancel"
                >
                  <Icon name="lucide:slash" class="size-4" />

                  <span class="sr-only">Cancel</span>
                </button>

                <button
                  class="border-muted-200 hover:border-primary-500 text-muted-700 dark:text-muted-200 hover:text-primary-600 dark:border-muted-700 dark:bg-muted-900 dark:hover:border-primary-500 dark:hover:text-primary-600 relative flex size-8 cursor-pointer items-center justify-center rounded-full border bg-white transition-colors duration-300"
                  type="button"
                  tooltip="Upload"
                >
                  <Icon name="lucide:arrow-up" class="size-4" />

                  <span class="sr-only">Upload</span>
                </button>

                <button
                  class="border-muted-200 hover:border-primary-500 text-muted-700 dark:text-muted-200 hover:text-primary-600 dark:border-muted-700 dark:bg-muted-900 dark:hover:border-primary-500 dark:hover:text-primary-600 relative flex size-8 cursor-pointer items-center justify-center rounded-full border bg-white transition-colors duration-300"
                  type="button"
                  tooltip="Remove"
                  @click="removeFileFromFileList(file.name)"
                >
                  <Icon name="lucide:x" class="size-4" />

                  <span class="sr-only">Remove</span>
                </button>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </BaseInputFileHeadless>
  </div>
</template>
