<template>
  <section class="container mt-5" v-if="!loading">
    <div class="row">
      <div class="col-md-8 m-auto">
        <div class="card bg-white py-2 px-4">
          <div class="card-body">
            <!-- back button -->
            <v-btn class="my-3 bg-color"  dark link to="/">
              <BaseIcon prop="fas fa-chevron-left mr-1" />Manage Courses</v-btn
            >

            <h4 class=" mb-4 text-secondary text-center">Edit Course</h4>

            <v-form
              ref="form"
              v-model="valid"
              @submit.prevent="handleBootcampCourseEdit"
            >
              <!-- title -->
              <v-text-field
                v-model="title"
                label="Course Title"
                :rules="titleRules"
                required
              ></v-text-field>

              <!-- duration -->
              <v-text-field
                v-model="duration"
                :rules="durationRules"
                label="Duration"
                hint="please enter number of weeks course lasts"
                required
              ></v-text-field>

              <!-- tuition -->
              <v-text-field
                v-model="tuition"
                :rules="tuitionRules"
                label="Tuition"
                hint="USD Currency"
                required
              ></v-text-field>

              <!-- skills requred -->
              <v-select
                v-model="minimumSkill"
                :items="items"
                chips
                label="Minimum Skill Required"
              ></v-select>

              <!-- description -->
              <v-textarea
                v-model="description"
                clearable
                clear-icon="cancel"
                :counter="500"
                required
                :rules="descriptionRules"
                label="Description"
                hint="No more than 500 characters"
              ></v-textarea>

              <v-checkbox
                v-model="scholarship"
                label="Scholarship Available"
                color="success"
                hide-details
              ></v-checkbox>

              <div class="form-group mt-4">
                <BaseButton name="Add Course" :loading="getLoading" />
              </div>
            </v-form>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import NProgress from 'nprogress';
import store from '@store/index';
import { mapGetters, mapActions } from 'vuex';
import LoadingMixin from '@mixins/LoadingMixins';

export default {
  mixins: [LoadingMixin],
  computed: mapGetters(['getErr', 'getOneCourse', 'getLoading']),
  data() {
    return {
      description: '',
      scholarship: false,
      minimumSkill: '',
      tuition: '',
      duration: '',
      title: '',
      valid: false,
      titleRules: [v => !!v || 'title is required'],
      items: ['beginner', 'intermediate', 'advance'],
      durationRules: [
        v => !!v || 'course duration is required',
        v => /^[0-9]*$/.test(v) || 'Only Numbers are allowed'
      ],
      tuitionRules: [
        v => !!v || 'course tuition is required',
        v => /^[0-9]*$/.test(v) || 'Only Numbers are allowed'
      ],

      descriptionRules: [
        value => !!value || 'Bootcamp Description is Required.',
        v => v.length <= 500 || 'Max 500 characters'
      ]
    };
  },
  methods: {
    ...mapActions(['getSingleCourse', 'updateCourse']),
    handleBootcampCourseEdit() {
      if (this.$refs.form.validate()) {
        this.snackbar = true;

        const data = {
          title: this.title,
          description: this.description,
          weeks: this.duration * 1,
          tuition: this.tuition * 1,
          minimumSkill: this.minimumSkill,
          scholarshipAvailable: this.scholarship,
          bootcamp: this.$route.params.bootId,
          id: this.$route.params.id
        };
        this.updateCourse(data).then(res => {
          if (res && res.data.success) {
            this.$noty.success('course updated successfully');
            this.$router.push(`/bootcamp/${this.$route.params.slug}`);
          } else {
            this.$noty.error(this.getErr);
          }
        });
      }
    }
  },
  created() {
    this.toggleLoading();
    NProgress.start();
    this.getSingleCourse(this.$route.params.id).then(res => {
      if (res && res.data.success) {
        this.description = this.getOneCourse.description;
        this.scholarship = this.getOneCourse.scholarshipAvailable;
        this.minimumSkill = this.getOneCourse.minimumSkill;
        this.tuition = this.getOneCourse.tuition;
        this.duration = this.getOneCourse.weeks;
        this.title = this.getOneCourse.title;

        NProgress.done();
        this.toggleLoading();
      }
    });
  }
};
</script>
