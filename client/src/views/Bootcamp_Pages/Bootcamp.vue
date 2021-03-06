<template>
  <section class="bootcamp mt-5" v-if="!loading && singleBootcamp">
    <div class="container">
      <div class="row">
        <!-- Main col -->
        <div class="col-md-8">
          <h2 class="text-secondary">{{ singleBootcamp.name }}</h2>

          <!-- Description -->
          <p>
            {{ singleBootcamp.description }}
          </p>
          <!-- Avg cost -->
          <p class="lead mb-4">
            Average Course Cost:
            <span class="text-primary"
              ><span>&#8358;</span> {{ singleBootcamp.averageCost }}</span
            >
          </p>
          <!-- Courses -->
          <div v-for="course in singleBootcamp.courses" :key="course._id">
            <div class="card mb-3">
              <div class="card-header course_bg text-white">
                <div class="d-flex justify-content-between align-items-center">
                  <!-- course title -->
                  {{ course.title }}
                  <!-- button to edit course -->
                  <div>
                    <router-link
                      v-if="
                        getAuthUser.role === 'admin' ||
                          getAuthUser._id === singleBootcamp.user
                      "
                      :to="
                        `/bootcamp/${course._id}/${singleBootcamp._id}/${singleBootcamp.slug}/bootcamp_course`
                      "
                    >
                      <v-tooltip bottom>
                        <template v-slot:activator="{ on }">
                          <v-chip v-on="on" link class="bg-color"
                            ><BaseIcon prop="fas fa-pencil-alt mr-1"
                          /></v-chip>
                        </template>
                        <span>Edit</span>
                      </v-tooltip>
                    </router-link>
                  </div>
                </div>
              </div>
              <div class="card-body">
                <div class="d-flex justify-content-between align-items-center">
                  <h5 class="card-title text-secondary">
                    Duration: {{ course.weeks }} Weeks
                  </h5>

                  <!-- display delete icon for admin and publisher only -->
                  <div>
                    <!-- delete bootcamp -->

                    <button
                      :disabled="loading"
                      v-if="
                        getAuthUser.role === 'admin' ||
                          getAuthUser._id === singleBootcamp.user
                      "
                      class="btn dark-bg mr-2"
                      @click="handleDeleteCourse(course._id)"
                    >
                      <BaseIcon
                        prop="fas fa-spin fa-spinner "
                        v-if="getLoading"
                      />
                      <BaseIcon prop="fas fa-trash " v-else />
                    </button>
                  </div>
                </div>
                <!-- bootcamp description -->
                <p class="card-text">
                  {{ course.description }}
                </p>
                <ul class="list-group mb-3">
                  <!-- course tuition -->
                  <li class="list-group-item">
                    Cost: <span>&#8358;</span> {{ course.tuition }} USD
                  </li>
                  <li class="list-group-item">
                    Skill Required: {{ course.minimumSkill }}
                  </li>
                  <li class="list-group-item">
                    Scholarship Available:
                    <span v-if="course.scholarshipAvailable"
                      ><BaseIcon prop="fas fa-check text-success mr-2"
                    /></span>

                    <span v-else
                      ><BaseIcon prop="fas fa-times text-danger mr-2"
                    /></span>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
        <!-- Sidebar -->
        <div class="col-md-4 ">
          <!-- Image -->
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
          <!-- Rating -->
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
          <BaseNormalBtn
            :route="
              `/bootcamp/${singleBootcamp._id}/${singleBootcamp.slug}/reviews`
            "
            ><BaseIcon prop="fas fa-comments mr-1" /> Read
            Reviews</BaseNormalBtn
          >
          <a
            :href="singleBootcamp.website"
            target="_blank"
            class="btn btn-block btn-color"
            ><BaseIcon prop="fas fa-globe mr-2" /><span class="text-light"
              >Visit Website</span
            >
          </a>

          <!-- Map  -->
          <div class="mt-12"><BootcampMap :bootcamp="singleBootcamp" /></div>
          <!-- Perks -->
          <v-card class="mx-auto mt-12" max-width="500" tile>
            <v-list disabled>
              <v-list-item-group color="primary">
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>
                      <span v-if="singleBootcamp.housing"
                        ><BaseIcon prop="fas fa-check text-success mr-3"
                      /></span>

                      <span v-else
                        ><BaseIcon prop="fas fa-times text-danger mr-3"
                      /></span>

                      Housing
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>
                      <span v-if="singleBootcamp.jobAssistance"
                        ><BaseIcon prop="fas fa-check text-success mr-3"
                      /></span>

                      <span v-else
                        ><BaseIcon prop="fas fa-times text-danger mr-3"
                      /></span>

                      Job Assistance
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>
                      <span v-if="singleBootcamp.jobGuarantee"
                        ><BaseIcon prop="fas fa-check text-success mr-3"
                      /></span>

                      <span v-else
                        ><BaseIcon prop="fas fa-times text-danger mr-3"
                      /></span>

                      Job Guarantee
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>
                      <span v-if="singleBootcamp.acceptGi"
                        ><BaseIcon prop="fas fa-check text-success mr-3"
                      /></span>

                      <span v-else
                        ><BaseIcon prop="fas fa-times text-danger mr-3"
                      /></span>
                      Accepts GI Bill
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </v-card>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import NProgress from 'nprogress';
import store from '@store/index';
import LoadingMixin from '@mixins/LoadingMixins';
import BootcampMap from '@bootcampsUtils/BootcampMap';

export default {
  mixins: [LoadingMixin],
  computed: mapGetters([
    'singleBootcamp',
    'getErr',
    'getAuthUser',
    'getLoading',
    'getPhoto'
  ]),
  components: {
    BootcampMap
  },
  methods: {
    ...mapActions(['deleteCourse', 'SingleBootcamps']),
    handleDeleteCourse(id) {
      this.deleteCourse(id).then(res => {
        if (res && res.data.success) {
          this.$noty.success('course deleted successfully');
          this.$router.push('/manage_bootcamp');
        } else {
          this.$noty.error(this.getErr);
        }
      });
    }
  },
  created() {
    this.toggleLoading();
    NProgress.start();
    this.SingleBootcamps(this.$route.params.slug).then(res => {
      if (res && res.data.success) {
        NProgress.done();
        this.toggleLoading();
      }
    });
  }
};
</script>

<style scoped>
.dark-bg,
.course_bg,
.btn-color {
  background: #181f44 !important;
}
.fas {
  color: #fff;
}

/* mobile phone screen sm <= 425px */
@media screen and (max-width: 425px) {
  h2.text-secondary {
    font-size: 25px;
    text-align: center;
    margin-bottom: 1rem;
  }
}
</style>
