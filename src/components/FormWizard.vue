<template>
  <div>
    <div v-if="wizardInProgress" v-show="asyncState !== 'pending'">
      <keep-alive>
        <component
          ref="currentStep"
          :is="currentStep"
          @updateAsyncState="updateAsyncState"
          :wizard-data="form"/>
      </keep-alive>

      <div class="progress-bar">
        <div :style="`width: ${progress}%;`"></div>
      </div>

      <!-- Actions -->
      <div class="buttons">
        <button
          @click="goBack"
          v-if="currentStepNumber > 1"
          class="btn-outlined"
        >Back
        </button>
        <button
          @click="nextBtnAction"
          class="btn">
          {{ isLastStep ? 'Complete Order' : 'Next' }}
        </button>
      </div>

      <pre><code>{{form}}</code></pre>
    </div>
    <div v-else>
      <h1 class="title">Thank you!</h1>
      <h2 class="subtitle">
        We look foward to shipping your first box
      </h2>

      <p class="text-center">
        <a href="https://vueschool.io" target="_blank" class="btn">Go somewhere cool</a>
      </p>
    </div>
    
    <div class="loading-wrapper" v-if="asyncState === 'pending'">
      <div class="loader">
        <img src="/spinner.svg" alt="">
        <p>Please wait, we're hitting our servers!</p>
      </div>
    </div>
  </div>
</template>

<script>
import {postFormToDB} from '../api'
import FormPlanPicker from './FormPlanPicker'
import FormUserDetails from './FormUserDetails'
import FormAddress from './FormAddress'
import FormReviewOrder from './FormReviewOrder'
import _ from 'lodash'
export default {
  name: 'FormWizard',
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data () {
    return {
      currentStepNumber: 1,
      asyncState: null,
      steps: [
        'FormPlanPicker',
        'FormUserDetails',
        'FormAddress',
        'FormReviewOrder'
      ],
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false
      }
    }
  },
  computed: {
    isLastStep () {
      return this.currentStepNumber === this.length
    },
    wizardInProgress () {
      return this.currentStepNumber <= this.length
    },
    currentStep () {
      return this.steps[this.currentStepNumber - 1]
    },
    length () {
      return this.steps.length
    },
    progress () {
      return this.currentStepNumber/this.length * 100
    }
  },
  methods: {
    updateAsyncState (state) {
      this.asyncState = state
    },
    submitOrder () {
      this.updateAsyncState('pending')
      postFormToDB(this.form)
        .then((returnedForm) => {
          console.log('form submitted', JSON.stringify(returnedForm))
          this.asyncState = 'success'
          this.currentStepNumber++
        })
    },
    nextBtnAction () {
      this.$refs.currentStep.submit()
        .then(stepData => {
          Object.assign(this.form, stepData)
          if (this.isLastStep) {
            this.submitOrder()
          } else {
            this.goNext()
          }
        })
        .catch(error => console.log(error))
    },
    goBack () {
      this.currentStepNumber--
    },
    goNext () {
      this.currentStepNumber++
    }
  }
}
</script>
