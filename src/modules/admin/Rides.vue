<template>
  <div v-if="data !== null" class="holder w-100">
    <!-- <input type="text" class="form-control" v-model="searchValue" placeholder="Search location" @keyup="filterLocation()"> -->
      <div class="row justify-content-end m-0 mr-5 align-items-center">
        <Pager
          :pages="numPages"
          :active="activePage"
          :limit="limit"
          />
      </div>
    <div class="row w-100 m-0">
      <div class="card card-half" v-for="(item, index) in result" :key="index" style="margin-bottom: 10px;" >
        <div>
          <div class="card-block px-3">
            <h6 class="card-title" style="margin-top:15px">
              {{readable(item.type)}}
            </h6>
            <h6 class="card-title " style="font-size: 15px; margin-top:15px; "><b>Model:</b> {{item.model ? item.model : 'Not Specified'}} <span class="ml-5"><b>Code:</b> {{item.code}}</span></h6>                            
            <span class="card-title">
              <b-button class="bg-black" style="margin-bottom: 25px; margin-bottom: 5px; " v-if="item.to_status === 'death'">
                DEATH
              </b-button>
              <b-button variant="danger" style="margin-bottom: 25px; margin-bottom: 5px; " v-if="item.to_status === 'positive'">
                POSITIVE
              </b-button>
              <b-button variant="warning" style="margin-bottom: 25px; margin-bottom: 5px;" v-if="item.to_status === 'pui'">
                PUI
              </b-button>
              <b-button variant="primary" style="margin-bottom: 25px; margin-bottom: 5px;" v-if="item.to_status === 'pum'"> 
                PUM
              </b-button>
              <b-button variant="success" style="margin-bottom: 25px; margin-bottom: 5px;" v-if="item.to_status === 'negative'">
               NEGATIVE
              </b-button>
            </span>
          </div> 
        </div>
      </div>
    </div>
  </div>
</template>
<style lang="scss" scoped> 
@import "~assets/style/colors.scss";
.custom-header-color{
  color: $primary;
}
.badge{
  margin-left: 5px;
}
.card-title{
  width: 100%;
  float: left;
}

.holder{
  float: left;
  margin-top: 25px;
}

.form-control{
  height: 45px !important;
  margin-bottom: 25px;
}

.bg-black{
  background: #000000 !important;
  color: #ffffff !important;
  border: 0px;
}
input[type="number"] {
  -webkit-appearance: textfield;
    -moz-appearance: textfield;
          appearance: textfield;
}
input[type=number]::-webkit-inner-spin-button, 
input[type=number]::-webkit-outer-spin-button { 
  -webkit-appearance: none;
}
.number-input {
    margin-bottom: 1rem;
    margin-left: 50rem;
    margin-right: 4rem;
    float: right;
}
.number-input button {
  -webkit-appearance: none;
  background-color: transparent;
  border: none;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  margin: 0;
  position: relative;
}
.number-input button:before,
.number-input button:after {
  display: inline-block;
  position: absolute;
  content: '';
  height: 2px;
  transform: translate(-50%, -50%);
}
.number-input button.plus:after {
  transform: translate(-50%, -50%) rotate(90deg);
}
.number-input input[type=number] {
  text-align: center;
}


.md-number-input.number-input {
  width: 11.8rem;
}
.md-number-input.number-input button {
  outline: none;
  width: 3rem;
  height: 2rem;
  padding-top: .8rem;
}
.md-number-input.number-input button.minus {
  padding-left: 8px;
}
.md-number-input.number-input button.plus {
  padding-left: 2px;
}
.md-number-input.number-input button:before,
.md-number-input.number-input button:after {
  width: 1rem;
  background-color: #212121;
}
.md-number-input.number-input input[type=number] {
  max-width: 5rem;
  padding: .5rem;
  border: solid #ddd;
  border-width: 0 2px;
  font-size: 2rem;
  height: 3rem;
  font-weight: bold;
  outline: none;
}

.card.card-half {
  width: 49%;
  margin: .5%;
}

@media (max-width: 767px) {
  .card.card-half {
    width: 100%;
    margin: 0;
  }
}

@media not all and (min-resolution:.001dpcm)
{ @supports (-webkit-appearance:none) and (stroke-color:transparent) {
  .number-input.md-number-input.safari_only button:before, 
  .number-input.md-number-input.safari_only button:after {
    margin-top: -.6rem;
  }
}}
</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import COMMON from 'src/common.js'
import Pager from 'src/components/increment/generic/pager/Pager.vue'
export default {
  mounted(){
    if(this.user.type !== 'ADMIN'){
      ROUTER.push('/dashboard')
    }
    this.retrieve()
  },
  data(){
    return {
      common: COMMON,
      user: AUTH.user,
      data: null,
      searchValue: null,
      result: null,
      limit: 10,
      activePage: 1,
      numPages: null
    }
  },
  components: {
    Pager,
    'google-map-modal': require('components/increment/generic/map/ModalMap.vue'),
    'increment-modal': require('components/increment/generic/modal/Modal.vue')
  },
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    retrieve(){
      let parameter = {
        status: 'positive',
        limit: this.limit,
        offset: this.activePage
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('rides/retrieve', parameter).then(response => {
        $('#loading').css({display: 'none'})
        if(response.data.length > 0){
          this.data = response.data
        }else{
          this.data = null
        }
        this.result = this.lists(this.data)
      })
    },
    readable(type){
      let readable = ''
      for(let i = 0; i < COMMON.vehicleTypes.length; i++) {
        let carType = COMMON.vehicleTypes[i]
        if(carType.value === type) {
          readable = carType.label
          break
        }
      }

      return readable === '' ? type : readable
    },
    linkGen (pageNum){
      return '#page=' + pageNum
    },
    lists(toFilter){
      let item = toFilter
      let pages = toFilter.length / this.limit
      let remaining = toFilter.length - (this.limit * pages)
      if (remaining > 0) {
        pages++
      }
      this.numPages = pages
      return item.slice(
        (this.activePage - 1) * this.limit,
        this.activePage * this.limit)
    }
  }
}
</script>
