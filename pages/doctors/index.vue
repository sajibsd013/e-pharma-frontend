<template>
  <section class="mb-5 mt-3 pb-5">
    <div class="banner_wrapper p-sm-5 p-3">
      <div class="container">
        <div class="my-3">
          <h1 class="text-light">Specialist Doctors</h1>
        </div>
        <div class="row g-3 mb-0">
          <div class="col-md-4">
            <div class="rounded bg-white">
              <div class="d-flex align-items-center px-2 rounded">
                <img
                  src="/img/icons/search.svg"
                  style="width: 18px"
                  alt=""
                  class="me-2"
                />
                <input
                  type="search"
                  class="form-control form-select-sm border-0 my-0"
                  placeholder="Search doctor by name"
                  v-model="name"
                />
              </div>
            </div>
          </div>
          <div class="col-md-4">
            <select
              class="form-select form-select-sm"
              aria-label="Default select example"
              v-model="specialty"
              required
            >
              <option value="" disabled selected>Select Speciality</option>
              <option value="all">All</option>
              <template v-for="(d, index) in getSpesialist">
                <option :value="d" :key="index">
                  {{ d }}
                </option>
              </template>
            </select>
          </div>
          <div class="col-md-4">
            <select
              class="form-select form-select-sm"
              aria-label="Default select example"
              required
              v-model="type"
            >
              <option value="" disabled selected>Select Service Type</option>
              <template v-for="(type, index) in s_type_list">
                <option :value="type" :key="index">
                  {{ type }}
                </option>
              </template>
            </select>
          </div>
        </div>
      </div>
    </div>
    <div class="container">
      <div class="row" v-if="sortedDoctorList.length">
        <h6 class="mt-2">{{ specialty == "all" ? "" : specialty }}</h6>
        <Doctors class="" :doctors="sortedDoctorList" />
      </div>
      <div v-else>
        <div class="text-center py-5">
          <h4>Nothing Found!</h4>
        </div>
      </div>
    </div>
    <div class="container" v-if="!isApp">
      <div v-for="(type, index) of s_type_list" :key="index">
        <DoctorSlider :doctors="get_doctors" :type="type" />
      </div>
    </div>
  </section>
</template>

<script>
import DoctorSlider from "~/components/Partials/DoctorSlider.vue";
export default {
  components: { DoctorSlider },
  head() {
    return {
      title: "স্বাস্থ্যসেবক || Doctor",
    };
  },
  data() {
    return {
      doctors: [],
      name: "",
      specialty: "",
      type: "",
      s_type_list: ["Private Chamber", "Video Call", "Home Call"],
    };
  },
  computed: {
    isApp() {
      return window.innerWidth < 767;
    },
    get_doctors() {
      return this.$store.getters["doctors/doctors"];
    },
    getSpesialistList() {
      return this.$store.getters["data/spesialist_list"];
    },
    sortedDoctorList() {
      if (!this.type && this.$route.query.stype) {
        this.type = this.$route.query.stype.replace("-", " ");
      }
      if (!this.specialty && this.$route.query.specialty) {
        let specialty_query = this.$route.query.specialty;
        specialty_query = specialty_query.replaceAll("-", " ");
        specialty_query = specialty_query.replaceAll("_", "/");
        this.specialty = specialty_query;
      }

      const filterd_data = this.get_doctors.filter(
        ({ status, specialty, type, name }) => {
          return (
            status == "approved" &&
            (this.name
              ? name.toUpperCase().includes(this.name.toUpperCase())
              : true) &&
            (this.type ? type.includes(this.type) : true) &&
            (this.specialty && this.specialty != "all"
              ? specialty == this.specialty
              : true)
          );
        }
      );
      return filterd_data;
    },
    getSpesialist() {
      const list = [];
      this.get_doctors.map(({ status, specialty }) => {
        if (status == "approved" && list.includes(specialty) == false) {
          list.push(specialty);
        }
        return;
      });

      return list.sort();
    },
  },

  methods: {
    // async getDoctors() {
    //   this.$nextTick(() => {
    //     this.$nuxt.$loading.start();
    //     this.$axios
    //       .get(`patners/doctor/`, {
    //         headers: {
    //           "Content-Type": "application/json",
    //         },
    //       })
    //       .then((res) => {
    //         if (res.status === 200) {
    //           this.doctors = res.data;
    //           this.$nuxt.$loading.finish();
    //         }
    //       })
    //       .catch((error) => {
    //         this.$nuxt.$loading.finish();
    //         console.log(error.response);
    //         console.log(error.response.data.message || error.message);
    //         // context.commit('error', error)
    //       });
    //   });
    // },
  },

  watch: {
    type(to, from) {
      this.$router.push("/doctors?stype=" + to.replace(" ", "-"));
    },
    specialty(to, from) {
      to = to.replaceAll(" ", "-");
      to = to.replaceAll("/", "_");
      this.$router.push("/doctors?specialty=" + to);
    },
  },
  mounted() {
    // this.getDoctors();
    this.$store.dispatch("data/getSpesialistList");
  },
};
</script>

<style scoped></style>
