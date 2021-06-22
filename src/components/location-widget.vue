<template>
  <div class="location-widget">
    <img
        :src="
        photoApi.web || require('@/assets/d4452d12509902c3a5b890f653f4fc19.jpg')
      "
        alt=""
    />
    <div class="text">{{ location }}</div>
  </div>
</template>

<script>
export default {
  name: "LocationWidget",
  props: {
    location: String,
  },
  data() {
    return {
      photoApi: Object,
    };
  },
  mounted() {
    this.getPhoto();
  },
  methods: {
    getPhoto() {
      fetch(
          "https://api.teleport.org/api/urban_areas/slug:" +
          this.location.trim().toLowerCase().replace(" ", "-") +
          "/images/"
      )
          .then((response) => {
            if (response.ok) {
              return response.json();
            } else {
              throw Error("Something went wrong...");
            }
          })
          .then((responseJson) => {
            if (responseJson.photos) {
              this.photoApi = responseJson.photos[0].image;
            }
          })
          .catch((error) => {
            console.log(error);
          });
    },
  },
};
</script>

<style lang="scss" scoped>
.location-widget {
  position: relative;

  img {
    border-radius: 25px;
    width: 100%;
    height: 100px;
    object-fit: cover;
    filter: brightness(80%);
  }

  .text {
    position: absolute;
    color: #fff;
    font-size: 1.5em;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -60%);
  }
}
</style>