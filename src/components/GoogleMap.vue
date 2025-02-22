<template>
    <div class="mb-4">
      <iframe
        width="100%"
        height="400"
        style="border:0;"
        loading="lazy"
        allowfullscreen
        :src="src"
        referrerpolicy="no-referrer-when-downgrade"
      ></iframe>
    </div>
  </template>

  <script>
  export default {
    name: 'GoogleMap',
    props: {
      src: {
        type: String,
        required: true
      }
    }
  };
  </script>
