<template>
    <div>
      <h2>Quản lý công việc</h2>
      <b-button v-b-modal.modal-prevent-closing style="background: rgb(74 176 33)">Thêm mới công việc</b-button>
      <b-button style="background: rgb(141 144 230)" @click="logout">Đăng xuất</b-button>
      <b-modal
        id="modal-prevent-closing"
        ref="modal"
        title="Thêm mới công việc"
        @show="resetModal"
        @hidden="resetModal"
        @ok="handleOk"
      >
        <form ref="form" @submit.stop.prevent="handleSubmit">
          <b-form-group
            label="Ngày"
            label-for="date-input"
            invalid-feedback="Bạn chưa chọn ngày"
            :state="job.dateState"
          >
            <b-form-input
              type="date"
              id="date-input"
              v-model="job.date"
              :state="job.dateState"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            label="Tiêu đề"
            label-for="title-input"
            invalid-feedback="Bạn chưa nhập tiêu đề"
            :state="job.titleState"
          >
            <b-form-input
              id="title-input"
              v-model="job.title"
              :state="job.titleState"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            label="Mô tả"
            label-for="description-input"
            invalid-feedback="Bạn chưa nhập mô tả"
            :state="job.descriptionState"
          >
            <b-form-input
              id="description-input"
              v-model="job.description"
              :state="job.descriptionState"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            label="Kết quả"
            label-for="result-input"
          >
            <b-form-input
              id="result-input"
              v-model="job.result"
            ></b-form-input>
          </b-form-group>
        </form>
      </b-modal>
      <b-modal
        id="modal-prevent-update"
        ref="modal"
        title="Thêm mới công việc"
        @hidden="resetModal"
        @ok="handleOk"
      >
        <form ref="form" @submit.stop.prevent="handleSubmit">
          <b-form-group
            label="Ngày"
            label-for="date-input"
            invalid-feedback="Bạn chưa chọn ngày"
            :state="job.dateState"
          >
            <b-form-input
              type="date"
              id="date-input"
              v-model="job.date"
              :state="job.dateState"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            label="Tiêu đề"
            label-for="title-input"
            invalid-feedback="Bạn chưa nhập tiêu đề"
            :state="job.titleState"
          >
            <b-form-input
              id="title-input"
              v-model="job.title"
              :state="job.titleState"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            label="Mô tả"
            label-for="description-input"
            invalid-feedback="Bạn chưa nhập mô tả"
            :state="job.descriptionState"
          >
            <b-form-input
              id="description-input"
              v-model="job.description"
              :state="job.descriptionState"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            label="Kết quả"
            label-for="result-input"
          >
            <b-form-input
              id="result-input"
              v-model="job.result"
            ></b-form-input>
          </b-form-group>
        </form>
      </b-modal>
      <TableJob :listJobs='listJobs'/>
    </div>
</template>

<script>
import axios from 'axios'
import {mapGetters} from 'vuex'
export default {
  async fetch () {
      const  data  = await axios.get('http://127.0.0.1:8000/api/job/list-job', {
             headers: {'Authorization': this.getCookie('token_authen')}
        })
      console.log(data)
      this.listJobs = data.data.data
    },
  computed: {
      ...mapGetters(['loggedInUser'])
  },
  data() {
      return {
        job: {
          id: '',
          title: '',
          description: '',
          date: '',
          result: '',
          titleState: null,
          descriptionState: null,
          dateState: null
        },
        listJobs: []
      }
  },
    methods: {
      getCookie(name) {
          const nameEQ = name + "=";
          const ca = document.cookie.split(';');
          for (let i = 0; i < ca.length; i++) {
            let c = ca[i];
            while (c.charAt(0) === ' ') c = c.substring(1, c.length);
            if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
          }
          return null;
      },
      async logout() {
        await this.$auth.logout();
        this.$axios.setToken(false);
        this.$router.push('/login');
      },
      checkFormValidity() {
        const valid = this.$refs.form.checkValidity()
        this.job.titleState = valid
        this.job.descriptionState = valid
        this.job.dateState = valid
        return valid
      },
      resetModal() {
        this.job.id = ''
        this.job.title = ''
        this.job.description = ''
        this.job.date = ''
        this.job.result = ''
        this.job.nameState = null
      },
      handleOk(bvModalEvt) {
        bvModalEvt.preventDefault()
        this.handleSubmit()
      },
      async handleSubmit() {
        if (!this.checkFormValidity()) {
          return
        }
        try {
         this.error = null
         await axios.post('http://127.0.0.1:8000/api/job/create-job', {
           title: this.job.title,
           description: this.job.description,
           date: this.job.date,
           result: this.job.result
         }, {
             headers: {'Authorization': this.$auth.getToken('local')}
        })
         axios.get(`http://127.0.0.1:8000/api/job/list-job`, {
             headers: {'Authorization': this.$auth.getToken('local')}
        })
        .then(response2 => {
          this.listJobs = response2.data.data
        })
        .catch(e => {
          this.errors.push(e)
        })
       } catch (error) {
           this.error = error.response.data
       }
        this.$nextTick(() => {
          this.$bvModal.hide('modal-prevent-closing')
        })
      },
    }
}
</script>
