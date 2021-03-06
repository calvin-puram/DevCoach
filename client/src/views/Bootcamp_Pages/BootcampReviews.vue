<template>
  <div>
    <section
      class="bootcamp mt-5"
      v-if="!loading && getBootcampReviews.length > 0"
    >
      <div class="container">
        <div class="row">
          <!-- Main col -->
          <div class="col-md-8 ">
            <v-btn
              class="my-3"
              color="#181f44"
              dark
              link
              :to="`/bootcamp/${singleBootcamp.slug}`"
              v-if="getBootcampReviews.length > 0 && singleBootcamp"
            >
              <BaseIcon prop="fas fa-chevron-left mr-1" />Bootcamp Info</v-btn
            >
            <h4
              class="my-5 text-secondary"
              v-if="getBootcampReviews.length > 0 && singleBootcamp"
            >
              {{ singleBootcamp.name }} Reviews
            </h4>

            <div v-for="bootcamp in getBootcampReviews" :key="bootcamp._id">
              <!-- Reviews -->

              <div class="card mb-3">
                <div
                  class="d-flex justify-content-between align-items-center card-header dark-bg text-white"
                >
                  <h5>
                    {{ bootcamp.title }}
                  </h5>
                  <div v-if="bootcamp.user._id === getAuthUser._id">
                    <router-link
                      :to="
                        `/bootcamp/${bootcamp._id}/${bootcamp.bootcamp.slug}/editReview`
                      "
                    >
                      <v-tooltip bottom>
                        <template v-slot:activator="{ on }">
                          <v-chip v-on="on" link color="teal"
                            ><BaseIcon prop="fas fa-pencil-alt mr-1"
                          /></v-chip>
                        </template>
                        <span>Edit</span>
                      </v-tooltip>
                    </router-link>
                  </div>
                </div>

                <div class="card-body">
                  <h5 class="card-title">
                    Rating:
                    <span class="text-success">{{
                      bootcamp.rating ? bootcamp.rating.toFixed(1) : 0
                    }}</span>
                  </h5>
                  <p class="card-text">
                    {{ bootcamp.text }}
                  </p>
                  <div
                    class="d-flex justify-content-between align-items-center"
                  >
                    <p class="text-muted">
                      Writtern By {{ bootcamp.user.name }}
                    </p>
                    <div v-if="bootcamp.user._id === getAuthUser._id">
                      <button
                        :disabled="getLoading"
                        class="btn dark-bg"
                        @click="deleteReview(bootcamp._id)"
                      >
                        <BaseIcon
                          prop="fas fa-spin fa-spinner "
                          v-if="getLoading"
                        />
                        <BaseIcon prop="fas fa-trash " v-else />
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <!-- Sidebar -->
          <div
            class="col-md-4 sidebar"
            v-if="getBootcampReviews.length > 0 && singleBootcamp"
          >
            <!-- Rating -->
            <img
              v-if="singleBootcamp.photo === 'no-photo.jpg'"
              :src="`/img/${singleBootcamp.photo}`"
              class="img-thumbnail"
              alt="bootcamp image"
            />
            <img
              v-else
              :src="`${singleBootcamp.photo}`"
              class="img-thumbnail"
              alt="bootcamp image"
            />

            <!-- rating -->
            <h3 class="text-center  my-4">
              <v-avatar color="orange" size="62">
                <span class="white--text headline">{{
                  singleBootcamp.averageRating
                    ? singleBootcamp.averageRating.toFixed(1)
                    : 0
                }}</span>
              </v-avatar>
              <span class="text-secondary ml-1">Rating</span>
            </h3>
            <!-- Buttons -->
            <!-- check if user has review to disable button -->
            <div>
              <!-- user has review -->
              <div v-if="checkUserCreateReview()">
                <v-btn
                  :disabled="checkUserCreateReview()"
                  class="my-3 bg-color"
                  block
                  link
                  :to="
                    `/bootcamp/${singleBootcamp._id}/${singleBootcamp.slug}/createReview`
                  "
                >
                  <BaseIcon prop="fas fa-pencil-alt mr-1" />Write Reviews</v-btn
                >
              </div>

              <!-- user has no review -->
              <div v-else>
                <BaseNormalBtn
                  :route="
                    `/bootcamp/${singleBootcamp._id}/${singleBootcamp.slug}/createReview`
                  "
                  ><BaseIcon prop="fas fa-pencil-alt mr-1" />Write
                  Reviews</BaseNormalBtn
                >
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- if no bootcamp reviews -->
    <section
      class="bootcamp mt-5"
      v-if="!loading && getBootcampReviews.length === 0"
    >
      <!-- back button -->

      <v-btn
        class="my-3 ml-5 bg-color"
        dark
        link
        :to="`/bootcamp/${singleBootcamp.slug}`"
      >
        <BaseIcon prop="fas fa-chevron-left mr-1" />Bootcamp Info</v-btn
      >
      <!-- content -->
      <h3 class="text-secondary text-center mt-12">
        {{ `So suprise that  ${singleBootcamp.name} has no review yet` }}
      </h3>

      <!-- write review btn -->

      <div class="text-center">
        <v-btn
          class="my-5 bg-color"
          dark
          link
          :to="
            `/bootcamp/${singleBootcamp._id}/${singleBootcamp.slug}/createReview`
          "
        >
          <BaseIcon prop="fas fa-pencil-alt mr-1" />Write Review</v-btn
        >
      </div>
    </section>
  </div>
</template>

<script>
import store from '@store/index';
import NProgress from 'nprogress';
import { mapGetters, mapActions } from 'vuex';
import LoadingMixin from '@mixins/LoadingMixins';

export default {
  computed: mapGetters([
    'getBootcampReviews',
    'singleBootcamp',
    'getErr',
    'getAuthUser',
    'getLoading'
  ]),
  mixins: [LoadingMixin],

  methods: {
    ...mapActions(['deletedReview', 'bootcampReviews', 'SingleBootcamps']),
    deleteReview(review) {
      this.deletedReview(review).then(res => {
        if (res && res.data.success) {
          this.$noty.success('Review deleted Successfully!');
          this.$router.push(`/bootcamp/${this.$route.params.slug}`);
        } else {
          this.$noty.error(this.getErr);
        }
      });
    },

    // check if user has create  review
    checkUserCreateReview() {
      const userIds = this.getBootcampReviews.map(review => review.user._id);
      const userHasReview = userIds.filter(
        user => user === this.getAuthUser._id
      );
      return userHasReview.length > 0;
    }
  },
  created() {
    NProgress.start();
    this.toggleLoading();
    this.bootcampReviews(this.$route.params.id).then(res => {
      this.SingleBootcamps(this.$route.params.slug).then(res => {
        if (res && res.data.success) {
          this.toggleLoading();
          NProgress.done();
        }
      });
    });
  }
};
</script>

<style scoped>
.sidebar {
  margin-top: 7rem;
}
.dark-bg {
  background: #181f44 !important;
}

.fas {
  color: #fff;
}
/* mobile phone screen sm <= 425px */
@media screen and (max-width: 425px) {
  .sidebar {
    margin-top: 1rem;
  }
}
</style>
