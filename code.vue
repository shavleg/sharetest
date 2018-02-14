<template>
  <div class="eleven wide computer sixteen wide mobile centered column">
    <div class="ui items">
      <div class="item">
        <!-- Avatar -->
        <!-- <div class="ui small circular image">
          <img :src="'https://avtorebi.com/img/ZsjhAUfRpalj9ncI.jpg?w=192&h=192&fit=crop'">
           <a href="#"><span class="edit icon"></span></a>
        </div> -->

                <!-- Avatar -->
        <div ref="cover"
          class="ui small circular image"

        >
          <!-- <div class="ui dimmer">
            <div class="content">
              <div class="center">
                <div @click="$refs.avatarSelect.click()" class="ui small purple button kaps fw-bold">
                  შეცვლა
                </div>
              </div>
            </div>
          </div> -->
          <img :src="'https://avtorebi.com/img/ZsjhAUfRpalj9ncI.jpg?w=192&h=192&fit=crop'">
          <div class="overlay"></div>
  <div class="button"><a @click="$refs.coverSelect.click()" > შეცვლა </a></div>
        </div>
        <input ref="coverSelect" type="file" accept="image/*" style="display: none;">

        <div class="content" style="margin: auto; text-align: ლეფტ;">
          <!-- Name -->
          <div class="ui large header" >
            {{ tags.list[0].name }}
          </div>

          <!-- Follow -->
          <follow-button
            v-if="isMe"
            :user="posts"
            :styles='{ "margin-left": "10px", "vertical-align": "super" }'
            @toggle="true"
          ></follow-button>

          <!-- About -->
          <div class="meta">
            <span>{{ tags.list[0].slug }}</span>
          </div>

          <!-- Author Types -->
          <!-- <div class="description">
            <span v-for="type in tags">
              {{ type.name }} &#9651;
            </span>
          </div> -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import FollowButton from "../../components/FollowButton.vue";

export default {
  components: { FollowButton },

  props: {
    posts: {
      type: Object,
      required: true
    },
    tags: {
      type: Object,
      required: true
    }
  },

  computed: {
    isMe() {
      return true; /*this.$store.state.user.id === this.tags.id*/
    }
  },
  methods: {
    updateCover(cover) {
      this.$progress.start();
      //UserService.updateAvatar(this.user.id, avatar)
      UserService.updateAvatar(1, cover)
        .then(() => this.fetchUser())
        .finally(() => this.$progress.done());
    }
  },
  mounted() {
    this.$progress.done();
    this.$nextTick(() => {
      $(this.$refs.coverSelect).on("change", evt => {
        alert();
        this.updateCover(evt.target.files[0]);
      });

      // this.setSelectedAuthorTypes(this.user.author_types)
    });
  }
};
</script>

<style>
.ui.small.image {
  position: relative;
  /* margin-top: 50px;*/
  width: 100px;
  height: 100px;
}

.ui.small.image .overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0);
  transition: background 0.5s ease;
}

.ui.small.image:hover .overlay {
  display: block;
  background: rgba(0, 0, 0, 0.3);
}

.ui.small.image img {
  position: absolute;
  width: 100px;
  height: 100px;
  left: 0;
}

.ui.small.image .button {
  position: absolute;
  width: 100px;
  left: 0;
  top: 40px;
  text-align: center;
  opacity: 0;
  transition: opacity 0.35s ease;
}

.ui.small.image .button a {
  /* width: 200px; */
  padding: 12px 4px;
  text-align: center;
  color: white;
  border: solid 2px white;
  z-index: 1;
  background-color: #a333c8;
}

.ui.small.image:hover .button {
  opacity: 1;
}
</style>
