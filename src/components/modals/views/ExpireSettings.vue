<template>
    <div class="modal-content fm-modal-about">
        <div class="modal-header">
            <h5 class="modal-title">{{ lang.modal.expiresettings.title }}</h5>
            <button type="button" class="close" aria-label="Close" v-on:click="hideModal">
                <span aria-hidden="true">&times;</span>
            </button>
        </div>
        <div class="modal-body">
            <dl class="row">
                <dt class="col-4">{{ lang.modal.expiresettings.current }}:</dt>
                <dd class="col-8">{{ currentExpiryString }}</dd>

                <dt class="col-4">{{ lang.modal.expiresettings.new }}:</dt>
                <dd class="col-8">
                    <input type="radio" v-model="expire_type" checked value="val"> {{ lang.modal.expiresettings.expires }}</radio>
                    <datepicker
                        name="expires_d"
                        v-model="expires_d"
                        :monday-first="true"

                        :format="'yyyy-MM-dd'"></datepicker><br>
                    <input type="radio" v-model="expire_type" value="never"> {{ lang.modal.expiresettings.never }}</radio></dd>
            </dl>
        </div>
        <div class="modal-footer">
            <button class="btn btn-danger" v-on:click="setExpire">{{ lang.modal.expiresettings.set }}
            </button>
            <button class="btn btn-light" v-on:click="hideModal">{{ lang.btn.cancel }}</button>
        </div>
    </div>
</template>

<script>
import modal from './../mixins/modal';
import translate from './../../../mixins/translate';
import helper from './../../../mixins/helper';
import Datepicker from 'vuejs-datepicker';

export default {
  name: 'ExpireSettings',
  mixins: [modal, translate, helper],
  components: { Datepicker },
  data() {
    return {
        expires_d: new Date(this.getExpiryTS() * 1000),
        expire_type: 'val'
    };
  },
  methods: {
    getExpiryTS() {
        var items = this.$store.getters['fm/selectedItems']
        if (items.every(elem => elem.expires == 0)) {
            return 0;
        }
        if (items.every(elem => elem.expires == null)) {
            return 0;
        }
        if (items.every(elem => elem.expires == items[0].expires)) {
            return items[0].expires;
        }
        else {
            // if we have mixed values, return the first non-zero value
            const firstnzval = (acc, el) => (acc > 0) ? acc : el.expires;
            return items.reduce(firstnzval);
        }
    },
    setExpire() {
      // create items list for delete
      const items = this.selectedItems.map(item => ({
        path: item.path,
        type: item.type,
      }));
      var expire_ts = 0;
      if (this.expire_type == 'val') {
        expire_ts = Date.parse(this.expires_d);
      }
      else {
        expire_ts = 0;
      }

      this.$store.dispatch('fm/setExpire', { files: items, expire: expire_ts / 1000}).then(() => {
          // close modal window
          this.hideModal();
      });
    },
  },
  computed: {
    /**
     * App version
     * @returns {*}
     */
     /**
      * Files and folders for deleting
      * @returns {*}
      */
     selectedItems() {
       return this.$store.getters['fm/selectedItems'];
     },
     currentExpiryString() {
        if (this.selectedItems.every(elem => elem.expires == 0)) {
            return 'Never';
        }
        if (this.selectedItems.every(elem => elem.expires == null)) {
            return 'Never';
        }
        if (this.selectedItems.every(elem => elem.expires == this.selectedItems[0].expires)) {
            return this.timestampToDate(this.selectedItems[0].expires);
        }
        else {
            return "Multiple dates";
        }
    },
  },
};
</script>
