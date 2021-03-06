<style lang="scss">
  .movie {

    &__header {
      position: relative;
      padding: 80px 0;
      overflow: hidden;
      display: flex;
      flex-flow: row nowrap;
      align-items: center;
      justify-content: center;
      box-shadow: 0 4px 8px #f0f1f2;
      background-image: linear-gradient(135deg, rgb(130, 130, 130) 0%, rgb(103, 103, 103) 75%);

      &__poster {

        &__image {
          overflow: hidden;
          max-height: 405px;
          max-width: 270px;
          box-shadow: 0 45px 100px rgba(0, 0, 0, 0.4);

          img {
            display: block;
            width: 100%;
            height: 100%;
            object-fit: cover;
          }
        }

        &__name {
          color: #fff;
          text-shadow: 0 0 10px rgba(0, 0, 0, .5);
          cursor: default;
          font-size: 1.5rem;
          padding: 0 10px;
          text-align: center;
          letter-spacing: 2px;
          @media ($screen-xs-max) {
            font-size: 1.3rem;
          }
        }
      }

      &__main {
        display: block;
        max-width: 400px;

        @media ($screen-xs-max) {
          margin-top: 30px;
        }

        &__item {
          font-size: 1.1rem;
          align-self: flex-start;
          letter-spacing: 2px;
          padding: 10px;
          cursor: default;
          color: #ffffff;
          text-shadow: 0 0 6px rgba(0, 0, 0, .4);

          @media ($screen-xs-max) {
            font-size: 1rem;
          }

          &:not(:first-child) {
            border-top: 1px solid rgba(255, 255, 255, .2);
          }
        }
      }

      @media ($screen-xs-max) {
        flex-flow: column nowrap;
      }

    }

  }
</style>

<template>
  <div class="movie">
    <div class="movie__header">
      <div class="movie__header__poster">
        <div class="movie__header__poster__image">
          <img
            :src="$getOssUrl(data.movieItem.image, true)"
            crossorigin="anonymous"
            @load="$gradientColor('.movie__header')"
          >
        </div>
        <Blocker height="20px" />
        <div class="movie__header__poster__name">
          {{ data.movieItem.name }}
          <template v-if="data.movieItem.memo">
            <small>{{ data.movieItem.memo }}</small>
          </template>
        </div>
      </div>
      <Blocker width="100px" />
      <div class="movie__header__main">
        <div v-if="data.movieItem.type" class="movie__header__main__item">
          <Icon name="category" />
          {{ data.movieItem.type }}
        </div>
        <div v-if="data.movieItem.score" class="movie__header__main__item">
          <Icon name="star" />
          {{ data.movieItem.score }}分
        </div>
        <div v-if="data.movieItem.people" class="movie__header__main__item">
          <Icon name="user" />
          {{ data.movieItem.people }}
        </div>
        <div v-if="data.movieItem.site" class="movie__header__main__item">
          <Icon name="location" />
          {{ data.movieItem.site }}
        </div>
        <div v-if="data.movieItem.datetime" class="movie__header__main__item">
          <Icon name="clock" />
          <Datetime :time="data.movieItem.datetime" type="date" from-now />
        </div>
      </div>
    </div>
    <template v-if="data.movieItem.scores">
      <Blocker height="60px" />
      <LayoutContainer class="movie__content">
        <Nameplate title="评分" sub-title="score">
          <Btn @click="status.showScoreModal = true">
            评个分
          </Btn>
        </Nameplate>
        <Tag>共 {{ data.movieItem.score_count }} 人参与评分</Tag>
        <Tag>总评为 {{ data.movieItem.score_total }} 分</Tag>
        <Tag v-for="item in data.movieItem.scores" :key="item.id">
          {{ item.name }} {{ item.score }} 分 -
          <Datetime :time="item.datetime" format="YYYY-MM-DD" from-now />
        </Tag>
      </LayoutContainer>
    </template>
    <template v-if="data.movieItem.schedule">
      <Blocker height="60px" />
      <LayoutContainer class="movie__content">
        <Nameplate title="追剧" sub-title="binge-watching" />
        <Tag>{{ data.movieItem.schedule }}</tag>
      </LayoutContainer>
    </template>
    <Blocker height="60px" />
    <LayoutContainer class="movie__content">
      <Nameplate title="想法" sub-title="thinking" />
      <Markdown :content="data.movieItem.comment || ''" />
    </LayoutContainer>
    <Blocker height="40px" />
    <LayoutContainer>
      <Comment :id="data.movieItem.id" module="movie" />
    </LayoutContainer>
    <Blocker height="60px" />
    <Modal v-model="status.showScoreModal" title="评个分" icon="star" width="500px">
      <Loading v-if="status.isLoadingSubmit" :fix="true">
        评分提交中
      </Loading>
      <FormItem
        v-model="form.name"
        validate="required|maxLength:15"
        label="评分人"
        name="name"
        type="input"
        placeholder="敢问大侠叫什么?"
        @changeValidate="valid => status.validate.name = valid"
      />
      <FormItem
        v-model="form.score"
        label="评分"
        name="score"
        type="input"
        validate="required|number|minValue:0.1|maxValue:10"
        placeholder="0.1-10"
        @changeValidate="valid => status.validate.score = valid"
      >
      </FormItem>
      <FormItem
        v-model="form.content"
        label="评语"
        name="content"
        type="input"
        validate="maxLength:60"
        placeholder="可简述为何这样评分"
        @changeValidate="valid => status.validate.content = valid"
      />
      <Btn slot="footer" :full-width="true" :colorful="true" height="48px" @click="submitMovieScore()">
        提交
      </Btn>
    </Modal>
  </div>
</template>

<script>
export default {
  validate ({ params }) {
    return /^\d+$/.test(params.id)
  },
  async asyncData ({ $axios, store, params }) {
    const { data: movieItem } = await $axios.$get(`/api/movies/${params.id}`)
    store.commit('SET_CURRENT_ITEM', { image: movieItem.image, name: movieItem.name, category: movieItem.category, date: movieItem.datetime })
    return {
      data: {
        movieItem
      }
    }
  },
  data () {
    return {
      status: {
        showScoreModal: false,
        isLoadingSubmit: false,
        validate: {
          name: false,
          score: false,
          content: true
        }
      },
      form: {}
    }
  },
  methods: {
    submitMovieScore () {
      if (this.$checkFormValidate(this.status.validate)) {
        this.status.isLoadingSubmit = true

        const newScore = {
          score: this.form.score,
          name: this.form.name,
          content: this.form.content
        }

        this.$axios.$put(`/api/movies/${this.data.movieItem.id}/score`, newScore)
          .then((response) => {
            this.data.movieItem.scores.push(response.data)
            this.form = {}
            this.status.showScoreModal = false
            this.status.isLoadingSubmit = false
          })
          .catch(() => {
            this.status.isLoadingSubmit = false
          })
      } else {
        this.$message.error('表单填写有误')
      }
    }
  },
  head () {
    const { name, category, description } = this.data.movieItem
    return {
      title: `${name} - ${category.name} - 观影`,
      meta: [
        {
          hid: 'index',
          name: 'description',
          content: this.$getSeoInfo('description', `${description || ''}`)
        }
      ]
    }
  }
}
</script>
