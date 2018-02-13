<template>
  <div class="ui fluid card" ref="comment">
    <div class="content">
      <!-- Avatar -->

      <!-- Name -->
      <router-link :to="{ name: 'author', params: { id: comment.author.id } }" class="header fw-normal" style="font-size: 15px; margin-top: 0;">
        {{ comment.author.name }}
      </router-link>
      <!-- Date -->
      <div class="meta">
        <span v-timeago="comment.created_at"></span>
      </div>
      <!-- Content -->
      <div class="description postComment">
        <div v-if="editMode" class="postComment--editor">
          <quill-editor
            v-model="editedComment"
            :config="editorConfig"
          ></quill-editor>
        </div>

        <div v-else v-html="comment.content" class="postComment--content"></div>
      </div>
    </div>

    <!-- Extra -->
    <div class="extra content">
      <div class="left floated">
        <div v-show="!editMode">
            <!-- Upvote -->
            <div @click="$click_vote"
                 :class="{ blue: comment.user_has.voted }"
                 class="tiny ui basic icon button kaps"
            >
                <i :class="{ remove: comment.user_has.voted, 'thumbs outline up': !comment.user_has.voted }"
                   class="icon"
                ></i> ხმის მიცემა {{ comment.total.votes }}
            </div>

            <!-- Edit -->
            <div v-if="isOwner"
                 @click="$click_edit"
                 class="tiny ui basic icon button kaps"
            >
                <i class="write icon"></i> რედაქტირება
            </div>

            <!-- Delete -->
            <div v-if="isOwner"
                 @click="$click_delete"
                 class="tiny ui basic icon button kaps"
            >
                <i class="remove icon"></i> წაშლა
            </div>

            <!-- Comments -->
            <div
                 @click="$click_comment_toogle"
                 class="tiny ui basic icon button kaps"
            >
                <i class="commenting icon"></i> კომენტარები
            </div>
        </div>
        <div v-show="editMode">
            <!-- Save -->
            <div @click="$click_save"
                 :class="{ loading: isSaving }"
                 class="tiny ui basic icon button kaps"
            >
                <i class="save icon"></i> შენახვა
            </div>

            <!-- Cancel -->
            <div @click="$click_cancel"
                 class="tiny ui basic icon button kaps"
            >
                <i class="remove icon"></i> გაუქმება
            </div>
        </div>
      </div>
    </div>
    <div class="social_comments" v-bind:id="'social_comments_'+comment.id">
 
      <div class="fb-comments"  v-if="ok" v-bind:data-href="href_data+'/'+comment.id" data-numposts="5"></div>
    </div>
  </div>
</template>

<script>
  import { quillEditor as QuillEditor } from 'vue-quill-editor'
  import { VoteService, CommentService } from '../../services'
  import { editorConfig } from './exports'
  import _ from 'underscore'

  export default {
    components: { QuillEditor },

    props: {
      comment: { type: Object, required: true },
      user: { type: Object, required: true },
      scroll: { type: Boolean, default: false }
    },

    data () {
      return {
        ok:false,
        isSaving: false,
        editMode: false,
        editorConfig: editorConfig,
        editedComment: '',
        href_data:document.location.href
      }
    },

    computed: {
      'isOwner': function () {
        return this.comment.author.id === this.user.id
      }
    },

    methods: {
      $click_vote () {
        this.comment.user_has.voted = !this.comment.user_has.voted

        VoteService.comment(this.comment.id, this.comment.user_has.voted)
          .then(() => {
            this.comment.total.votes += this.comment.user_has.voted ? 1 : -1
          }, () => {
            this.comment.user_has.voted = !this.comment.user_has.voted
          })
      },

      $click_edit () {
        this.editedComment = this.comment.content
        this.editMode = true
      },

      $click_comment_toogle()
      {
        var el = '#social_comments_'+this.comment.id;
        var clicks = $(this).data('clicks');
        if (!clicks) {
          $(el).animate({height:'300px'},500, function(){$(el).css('overflow','auto');});

        } else {
          $(el).animate({height:'0'},500, function(){$(el).css('overflow','hidden');});
        }
        $(this).data("clicks", !clicks);
      },

      $click_cancel () {
        this.editMode = false
        this.editedComment = '';
      },

      $click_save () {
        if (_.isEmpty(this.editedComment)) {
          return
        }

        this.isSaving = true
        CommentService.update(this.comment.id, this.editedComment)
          .then(response => {
            this.$emit('update', response.data)
          })
          .finally(() => {
            this.isSaving = false
            this.$click_cancel()
          })
      },

      $click_delete () {
        this.$bus.$emit('modal:confirm', this.$lang.comment.deleteMessage, response => {
          if (response === 'ok') {
            CommentService.delete(this.comment.id)
              .then(() => {
                this.$emit('delete', this.comment)
              })
          }
        })
      }
    },

    mounted () {

      this.$nextTick(function () {
  this.ok=true;
})

      //$('.fb-comments').attr('data-href',document.location.href);
      // Scroll to comment
      if (this.scroll) {
        setTimeout(() => {
          $('html, body').animate({
            scrollTop: $(this.$refs.comment).offset().top
          }, 500)
        }, 500)
      }
    }
  }
</script>

<style>
.postComment--editor .ql-toolbar.ql-snow,
.postComment--editor .ql-container.ql-snow {
  border: none !important;
}
.social_comments
{
  height: 0px;
  overflow: hidden;
}
</style>
