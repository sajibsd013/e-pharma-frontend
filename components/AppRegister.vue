<template>
  <section class="mb-5 pb-5">
    <div class="col-lg-4 col-md-6 col-12 mx-auto py-3">
      <img
        class="mx-auto d-block"
        src="/img/icons/login.jpeg"
        width="240"
        height="240"
        alt=""
      />

      <div class="p-3 card border-0 bg-white">
        <div class="sign-info">
          <h5 class="mb-3 small">Register now to explore more!</h5>
        </div>
        <ValidationObserver v-slot="{ handleSubmit }">
          <form class="form" @submit.prevent="handleSubmit(sendOTP)">
            <div class="input-field pb-1">
              <div class="input-group flex-nowrap">
                <span
                  class="input-group-text bg-white text-center"
                  style="width: 73px"
                  id="addon-wrapping"
                >
                  <img
                    src="@/static/img/icons/user_icon.svg"
                    alt="user icon"
                    width="37"
                    class="d-block mx-auto"
                  />
                </span>
                <input
                  type="text"
                  required
                  class="form-control py-2"
                  placeholder="Enter your name"
                  aria-describedby="addon-wrapping"
                  v-model="form_data.full_name"
                />
              </div>
            </div>
            <div class="input-field pb-1">
               <label for="" style="font-size:12px;" class="form-label small fw-semibold text-muted mb-0"
                >Type Your Number Without 0</label
              >
              <div class="input-group flex-nowrap">

                <span
                  class="input-group-text bg-white p-0"
                  style="width: 73px"
                  id="addon-wrapping"
                >
                  <select
                    class="form-select form-select-sm border-0 pe-0"
                    style="font-weight: 500"
                    aria-label=".form-select-sm example"
                    v-model="form_data.country_code"
                  >
                    <!-- <template v-for="(c, index) in CountryCode">
                      <option :value="c.dial_code" :key="index">
                        {{ c.dial_code }}
                      </option>
                    </template>
                    <template v-if="CountryCode.length">
                      <option value="+880">+880</option>
                    </template> -->
                    <option value="+880">+880</option>
                  </select>
                </span>

                <input
                  type="number"
                  required
                  class="form-control py-2"
                  placeholder="1XXXXXXXXX"
                  aria-label="phone"
                  aria-describedby="addon-wrapping"
                  v-model="form_data.phone"
                />
              </div>
            </div>
            <div>
              <div
                class="d-flex align-items-center p-0 my-1"
                v-if="!form_data.otp_status"
              >
                <input
                  class="p-0 m-0"
                  type="checkbox"
                  id="checkbox"
                  v-model="form_data.terms"
                  required
                />
                <p for="checkbox" class="py-0 my-0 ms-1 small">
                  Accept the
                  <a href="#" class="text-primary text-decoration-none"
                    >Terms</a
                  >
                  and
                  <a href="#" class="text-primary text-decoration-none"
                    >Privacy Policy</a
                  >
                </p>
              </div>

              <div class="d-grid gap-2">
                <button
                  class="btn btn-primary btn-sm"
                  :disabled="
                    form_data.phone.length != 10 ||
                    form_data.count_down > 0 ||
                    disable_btn
                  "
                  type="submit"
                >
                  <template v-if="form_data.count_down > 0">
                    Send again {{ form_data.count_down }} seconds later
                  </template>
                  <template v-else>
                    {{
                      form_data.otp_status
                        ? "Resend Verification Code "
                        : "Send Verification Code"
                    }}
                  </template>
                </button>
              </div>
            </div>
          </form>
        </ValidationObserver>
        <ValidationObserver
          v-slot="{ handleSubmit }"
          v-if="form_data.otp_status"
        >
          <form class="form my-4" @submit.prevent="handleSubmit(submitOTP)">
            <div class="input-field pb-3">
              <div class="input-group flex-nowrap">
                <span
                  class="input-group-text bg-white text-center"
                  style="width: 73px"
                  id="addon-wrapping"
                >
                  OPT
                </span>
                <input
                  type="number"
                  required
                  class="form-control py-2"
                  placeholder="Enter OTP"
                  aria-describedby="addon-wrapping"
                  v-model="form_data.otp"
                />
              </div>
            </div>

            <div class="d-grid gap-2">
              <button
                class="btn btn-primary btn-sm"
                :disabled="
                  !form_data.otp || form_data.otp.length != 4 || disable_btn2
                "
                type="submit"
              >
                Submit Verification Code
              </button>
            </div>
          </form>
        </ValidationObserver>

        <div class="register-footer text-center pt-2">
          <NuxtLink to="/login" class="small text-decoration-none text-dark"
            >Already have an account?
            <span class="text-primary text-decoration-underline">
              Sign In
            </span>
          </NuxtLink>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { ValidationObserver, ValidationProvider } from "vee-validate";

export default {
  name: "register",
  components: {
    ValidationObserver,
    ValidationProvider,
  },
  data() {
    return {
      form_data: {
        phone: "",
        full_name: "",
        country_code: "+880",
        terms: false,
        type: "register",
        otp: "",
        otp_status: false,
        count_down: 0,
      },
      disable_btn: false,
      disable_btn2: false,
    };
  },
  computed: {
    CountryCode() {
      return this.$store.getters["CountryCode"];
    },
    regiForm() {
      return this.$store.getters["auth_state/RegiForm"];
    },
  },

  methods: {
    countDownTimer() {
      if (this.form_data.count_down > 0) {
        setTimeout(() => {
          this.form_data.count_down -= 1;
          // this.$store.commit(" auth_state/setRegiForm", { ...this.form_data });
          this.countDownTimer();
        }, 1000);
      }
    },
    async sendOTP() {
      this.$nextTick(() => {
        this.$nuxt.$loading.start();
        this.disable_btn = true;
        const data = {
          phone: this.form_data.country_code + this.form_data.phone,
          type: this.form_data.type,
        };
        try {
          this.$axios
            .post(`sent-otp/`, data, {
              headers: {
                "Content-Type": "application/json",
                //  Authorization: "Bearer " + this.$config.apiToken2,

              },
            })
            .then((res) => {
              console.log(res);
              if (res.status === 201) {
                console.log(res.data);
                this.$toast.info("Please enter verification code... ");
                this.form_data.otp_status = true;
                this.form_data.count_down = 60;
                this.countDownTimer();
                this.$nuxt.$loading.finish();
              }
              this.$nuxt.$loading.finish();
            })
            .catch((error) => {
              console.log(error);
              this.$nuxt.$loading.finish();

              console.log(error.message || error.response.data.message);
            });
        } catch (e) {
          this.$nuxt.$loading.finish();

          console.log(e.response);
          this.$toast.error("Error Found! Try again...");
        }
        this.disable_btn = false;
      });
    },

    async submitOTP() {
      this.disable_btn2 = true;

      const data = {
        phone: this.form_data.country_code + this.form_data.phone,
        full_name: this.form_data.full_name,
        otp: this.form_data.otp,
      };

      try {
        const response = await this.$axios.post("auth/register", data);
        if (response.status === 201) {
          try {
            const response = await this.$auth.loginWith("local", {
              data: data,
            });
            if (response.status === 200) {
              this.$toast.success("Successfully created");
            } else {
              this.$toast.error("OTP Not Mathed!");
            }
          } catch (e) {
            this.$toast.error("Error Found! Try again...");
          }
        } else {
          this.$toast.error("Error Found! Try again...");
        }
      } catch (e) {
        if (e.response && e.response.data) {
          const errors = e.response.data;
          if (errors.phone && errors.phone[0]) {
            this.$toast.error("User already exists");
          }
        } else {
          this.$toast.error("Registation failed! Try again...");
        }
      }
      this.disable_btn2 = false;
    },
  },
  mounted() {
    // this.CountryCode = this.CountryCode.length
    //   ? this.CountryCode
    //   : this.$store.dispatch("getCountryCodes");

    const v_form_data = this.regiForm;
    v_form_data ? (this.form_data = { ...v_form_data }) : "";
    this.form_data.count_down > 0 ? this.countDownTimer() : "";
  },
  beforeCreate() {
    if (this.$auth.$state.loggedIn) {
      this.$router.push("/");
    }
  },
};
</script>
