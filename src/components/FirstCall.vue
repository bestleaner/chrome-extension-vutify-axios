<template>
  <v-card tile flat height="100%">
    <v-card-text>
      <v-form class="mx-4 mx-sm-12 pb-8" ref="form" v-model="valid" lazy-validation>
        <v-text-field
          v-model="userName"
          :rules="userNameRules"
          :label="'User Name'"
          required
        ></v-text-field>
        <v-text-field
          v-model="password"
          :rules="passwordRules"
          :label="'Password'"
          :type="'password'"
          required
        ></v-text-field>
        <v-text-field
          v-model="merchantNumber"
          :rules="merchantNumberRules"
          :label="'Merchant Number'"
          required
        ></v-text-field>
      </v-form>
      <v-card-actions>
        <v-btn :disabled="!valid" color="blue" class="mr-4" @click="submit">
          Submit
        </v-btn>
        <v-card-text>{{ message }}</v-card-text>
      </v-card-actions>
    </v-card-text>
  </v-card>
</template>

<script>
import axios from 'axios'

export default {
  name: 'FirstCall',
  data () {
    return {
      valid: true,
      userName: '',
      userNameRules: [
        v => !!v || 'Name is required',
        v => (v && v.length <= 10) || 'Name must be less than 10 characters'
      ],
      password: '',
      passwordRules: [
        v => !!v || 'Password is required',
        v => (v && v.length >= 1) || 'Password must be valid'
      ],
      merchantNumber: '',
      merchantNumberRules: [
        v => !!v || 'Merchant Number is required',
        v => (v && v.length >= 1) || 'Merchant Number must be less than 10 characters'
      ],
      message: ''
    }
  },
  mounted () {
    browser.runtime.sendMessage({})
  },
  computed: {
    defaultText () {
      return browser.i18n.getMessage('extName')
    }
  },
  methods: {
    submit () {
      if (this.$refs.form.validate()) {
        this.getMerchantId()
      }
    },
    async getMerchantId () {
      const axiosConfig = this.buidAxiosConfig({
        headers: {
          Username: this.userName,
          Password: this.password
        },
        body: {}
      })

      try {
        const response = await axios.get(process.env.VUE_APP_WARPLITE_API_SERVER + '/v2/merchants?merchantnumber=like' + this.merchantNumber + '&fields=merchantid,merchantnumber', axiosConfig)
        this.postMerchant(response.data.data[0].merchantid)
      } catch (error) {
        alert('First method(get merchant id) call failed: ' + error)
      }
    },

    async postMerchant (merchantId) {
      const axiosConfig = this.buidAxiosConfig({
        headers: {
          Username: this.userName,
          Password: this.password
        },
        body: {
          portaluser: this.userName,
          portalpass: this.password,
          merchantid: merchantId
        }
      })
      try {
        const response = await axios.post(process.env.VUE_APP_WARPLITE_API_SERVER + '/v2/portals', axiosConfig.body, axiosConfig)
        this.$refs.form.reset()
        this.message = 'Submitted successfully, message:' + response.data.summary.message.status
      } catch (error) {
        alert('Second method(submit merchant) call failed: ' + error)
      }
    },

    buidAxiosConfig ({ headers, body }) {
      return {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8',
          'Access-Control-Allow-Origin': '*',
          Authorization: process.env.VUE_APP_WARPLITE_API_TOKEN,
          ...headers
        },
        body
      }
    }
  }
}
</script>

<style scoped>
p {
  font-size: 20px;
}
</style>
