<template>
  <div class="bg-gray overflow-hidden border-bottom">
    <div class="container-md py-5 p-responsive overflow-hidden">
      <div class="float-right">
        <a :href="'obyte:' + address" class="btn">
          <svg height="16" width="16" class="mr-2 v-align-middle">
            <circle cx="8" cy="8" r="7" stroke="#ccc" stroke-width="1" fill="white" />
          </svg>
          Send bytes
        </a>
      </div>
      <div class="float-left mr-3">
        <Avatar size="100" :address="address"/>
      </div>
      <h2>
        {{profile.name || getAddressName(address) || obyteUsername || 'Undefined'}}
        <span v-if="getVerifiedStatus(address)" class="tooltipped tooltipped-n" aria-label="Verified">
          <span class="octicon octicon-verified mb-1"></span>
        </span>
      </h2>
      <div class="mb-1">
        <!--<span class="octicon octicon-zap mr-1"></span>-->
        <a :href="'https://explorer.obyte.org/#' + address" target="_blank">
          <span class="monospace">{{address}}</span>
        </a>
        <span class="octicon octicon-link-external f3 ml-2"/>
      </div>
      <div>
        <span v-if="profile.location" class="mr-2">
          <span class="octicon octicon-location mr-1"></span>{{profile.location}}
        </span>
        <span v-if="profile.website" class="mr-2">
          <span class="octicon octicon-link mr-1"></span><a :href="profile.website" target="_blank">{{profile.website}}</a>
        </span>
        <span v-if="email" class="mr-2">
          <span class="octicon octicon-mail mr-1"></span><a :href="'mailto:' + email" target="_blank">{{email}}</a>
        </span>
        <span v-if="steemUsername" class="mr-2">
          <span class="iconfont icon-steem mr-1"></span>
          <a :href="'https://steemit.com/@' + steemUsername" target="_blank">
            <span>{{steemUsername}}</span>
          </a>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import utils from '@/helpers/utils';
import { mapActions } from 'vuex';

export default {
  props: ['address', 'attestations'],
  computed: {
    profile() {
      return this.$store.state.profiles[this.address]
        && this.$store.state.profiles[this.address].profile
        ? this.$store.state.profiles[this.address].profile
        : {};
    },
    obyteUsername() {
      let obyteUsername = null;
      if (this.attestations.isLoaded) {
        Object.keys(this.attestations.messages).forEach((i) => {
          const attestation = this.attestations.messages[i];
          if (!obyteUsername && attestation.unit_authors.includes('UENJPVZ7HVHM6QGVGT6MWOJGGRTUTJXQ') && attestation.payload.profile.username) {
            obyteUsername = attestation.payload.profile.username;
          }
        });
      }
      return obyteUsername;
    },
    steemUsername() {
      let steemUsername = null;
      if (this.attestations.isLoaded) {
        Object.keys(this.attestations.messages).forEach((i) => {
          const attestation = this.attestations.messages[i];
          if (!steemUsername && attestation.unit_authors.includes('JEDZYC2HMGDBIDQKG3XSTXUSHMCBK725') && attestation.payload.profile.steem_username) {
            steemUsername = attestation.payload.profile.steem_username;
          }
        });
      }
      return steemUsername;
    },
    email() {
      let email = null;
      if (this.attestations.isLoaded) {
        Object.keys(this.attestations.messages).forEach((i) => {
          const attestation = this.attestations.messages[i];
          if (!email && attestation.unit_authors.includes('H5EZTQE7ABFH27AUDTQFMZIALANK6RBG') && attestation.payload.profile.email) {
            // eslint-disable-next-line prefer-destructuring
            email = attestation.payload.profile.email;
          }
        });
      }
      if (this.profile.email) {
        // eslint-disable-next-line prefer-destructuring
        email = this.profile.email;
      }
      return email;
    },
  },
  methods: {
    getVerifiedStatus: address => utils.getVerifiedStatus(address),
    getAddressName: address => utils.getAddressName(address),
    ...mapActions([
      'getProfile',
    ]),
  },
  created() {
    this.getProfile(this.address);
  },
};
</script>

<style lang="less">
@import '../../vars';

.octicon-verified {
  color: @primary-color;
}
</style>
