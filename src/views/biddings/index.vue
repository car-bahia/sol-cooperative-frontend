<style scoped lang="scss">
  .list-item {
    border-bottom: none;
  }

  .badge {
    &.draft,
    &.closed,
    &.waiting {
      background-color: $greyish-brown;
    }

    &.ongoing {
      background-color: $cerulean;
    }

    &.canceled,
    &.failure {
      background-color: $danger-color;
    }

    &.suspended,
    &.under_review {
      background-color: $peach;
    }

    &.approved,
    &.finnished,
    &.accepted {
      background-color: $apple-green;
    }
  }

</style>


<template lang="pug">
  .root.mt-2
    .text-center.container
      router-link.button.button-primary.router-link.mb-0.button-new-bidding.u-full-width(:to="{ name: 'newBidding' }")
        | {{ $t('.button.new') }}

    form.mt-2.container.filter(ref="form", method="get" @submit.prevent="fetchSearch")
      .search-field
        input-field(
          type="text",
          v-model="search",
          name="search",
          :placeholder="$t('.search.placeholder')",
          :hideLabel='true'
        )

    div(v-if="biddingsCount > 0  && !searching")
      ul
        li.list-item.row(v-for="(bidding) in biddings")
          router-link(:to="{ name: 'bidding', params: { id: bidding.id } }")
            .container
              .card
                .list-text
                  h5.mb-1 {{ bidding.title }}

                  div
                    label.mr-0-5.inline-block
                      | {{ $t('models.bidding.attributes.status') }}:
                    span.badge(:class="bidding.status")
                      | {{ $t('models.bidding.attributes.statuses.' + bidding.status) }}

                  div
                    label.mr-0-5.inline-block
                      | {{ $t('.period') }}:
                    | {{ $l('date.formats.default', bidding.start_date) }} - {{ $l('date.formats.default', bidding.closing_date) }}

                  div
                    label.mr-0-5.inline-block
                      | {{ $t('models.bidding.attributes.covenant') }}:
                    | {{ bidding.covenant_name }}

                  div
                    label.mr-0-5.inline-block
                      | {{ $t('models.bidding.attributes.classificationId') }}:
                    | {{ bidding.classification_name }}

                  hr.mt-1.mb-1
                  div
                    | {{ bidding.description }}

    .text-center(v-else-if="biddingsCount == 0")
      | {{ $t('.empty') }}

    .blink-50.big(v-else)
      ul
        li.list-item.mb-1
          .container
            .card
              .list-title
              i.fas.fa-spinner.fa-pulse.fa-2x
        li.list-item.mb-1.loading-2
          .container
            .card
              .list-title
              i.fas.fa-spinner.fa-pulse.fa-2x

        li.list-item.mb-1.loading-3
          .container
            .card
              .list-title
              i.fas.fa-spinner.fa-pulse.fa-2x

        li.list-item.mb-1.loading-4
          .container
            .card
              .list-title
              i.fas.fa-spinner.fa-pulse.fa-2x

    .row
      .twelve.columns
        paginator(
          v-model="page",
          :prev="prevPageLink",
          :next="nextPageLink",
          v-if="biddingsCount"
        )

</template>

<script>
  import parseLinkHeaders from 'parse-link-header'

  export default {
    name: 'biddings',
    data () {
      return {
        i18nScope: 'biddings.index',

        // resource
        biddings: null,
        biddingsCount: null,
        params: {},

        tabs: [],

        //search
        search: '',
        searching: false,

        // pagination
        page:          1,
        prevPageLink:  '',
        nextPageLink:  '',
        totalPages:    1,
      }
    },

    computed: {
      fetchParams() {
        return this.params
      }
    },

    methods: {

      formatDate: function(value) {
        if (value) {
          let newValue = value.split('/')
          return newValue[2] + "-" + newValue[1] + "-" + newValue[0]
        }
      },

      fetch() {
        let params = this.params

        return this.$http.get('/cooperative/biddings', { params })
          .then((response) => {
            this.biddings = response.data
            this.biddingsCount = this.biddings.length

            this.updatePagination(response)

            this.searching = false
          }).catch((_err) => {
            this.error = _err
            console.error(_err)
          })
      },

      updatePagination(aResponse) {
        this.page = aResponse.headers['x-page']
        this.totalPages = aResponse.headers['x-total']
        let links = parseLinkHeaders(aResponse.headers.link) || {}

        this.prevPageLink = _.dig(links, 'prev', 'page')
        this.nextPageLink = _.dig(links, 'next', 'page')
      },

      fetchSearch() {
        this.searching = true

        this.params = { search: this.search }

        this.updateRoute()
      },

      updateRoute() {
        let data = Object.assign({}, this.$route.query, this.params);

        this.$router.push({ name: 'biddings', query: data });
      },

      init() {
        this.params = this.$route.query
      }
    },

    created: function () {
      this.$emit('tabChanged', this.tabs)
      this.$emit('navbarTitleChanged', this.$t('.title'))

      this.init();
    },

    watch: {
      fetchParams() {
        this.fetch()
      },

      page() {
        this.params = Object.assign({}, this.params, { page: this.page });
        this.updateRoute()
      }
    }
  }

</script>
