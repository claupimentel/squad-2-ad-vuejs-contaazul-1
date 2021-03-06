<template>
  <section class="error-single">
    <div class="columns is-mobile is-marginless is-multiline error-header">
      <div class="column is-6">
        <div class="big-title-style page-header--title is-pulled-left">Detalhe do Erro</div>
      </div>
      <div class="column is-6">
        <router-link to="/">
          <button class="button is-info is-outlined is-pulled-right">
            <span class="icon is-small">
              <i class="fas fa-arrow-left"></i>
            </span>
            <span>Voltar</span>
          </button>
        </router-link>
      </div>
    </div>
    <loading-page v-if="isLoading" key="loading"></loading-page>
    <div v-else class="error-detail" :class="classMessage">
      <div class="columns is-mobile is-multiline">
        <div class="column error-detail--content">
          <h1 class="medium-title-style">
            <span class="is-size-2">{{ error.name }}</span>
            <span class="error-detail--at">@</span>
            {{ error.origin }}
          </h1>
          <div class="subtitle date error-detail--content-date">
            <span class="icon">
              <i class="far fa-clock"></i>
            </span>
            <span>{{ error.last_date | formatDate }}</span>
          </div>
          <div class="error-detail--content-line">
            <div class="small-title-style">
              <span class="icon">
                <i class="far fa-star"></i>
              </span>
              <span>Título</span>
            </div>
            <div class="text">{{ error.title }}</div>
          </div>
          <div class="error-detail--content-line">
            <div class="small-title-style">
              <span class="icon">
                <i class="fas fa-code"></i>
              </span>
              <span>Detalhes</span>
            </div>
            <div class="text" v-for="(detail, index) in error.details" :key="index">
              {{ detail }}
            </div>
          </div>
        </div>
        <div class="column column-message is-one-fifth-desktop">
          <div class="message">
            <div class="buttons has-addons">
              <button
                v-if="!error.archived"
                class="button is-primary is-outlined"
                @click="archiveItem"
              >
                <span class="icon"><i class="fa fa-archive"></i></span>
                <span>Arquivar</span>
              </button>
              <button class="button is-danger is-outlined" @click="deleteItem">
                <span class="icon">
                  <i class="fa fa-trash-alt"></i>
                </span>
                <span>Excluir</span>
              </button>
            </div>
            <div class="message-body" :class="classMessage">
              <div class="tag is-medium" :class="classMessage">
                <span class="icon">
                  <i class="fas fa-hashtag"></i>
                </span>
                <span>{{ error.level }}</span>
              </div>
              <div class="label-style">
                <span class="icon">
                  <i class="fas fa-clipboard-list"></i>
                </span>
                <span>Eventos</span>
              </div>
              <div class="small-title-style">{{ error.qty }}</div>
              <div class="label-style">
                <span class="icon">
                  <i class="fas fa-user"></i>
                </span>
                <span>Usuário</span>
              </div>
              <div class="small-title-style">{{ error.collected_by }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import LoadingPage from '@/components/LoadingPage.vue';

export default {
  components: {
    LoadingPage,
  },
  props: {
    id: String,
  },
  data() {
    return {
      error: {},
      isLoading: false,
    };
  },
  computed: {
    ...mapGetters(['getSelectedItems']),
    classMessage() {
      return this.error.level;
    },
  },
  methods: {
    ...mapActions(['setSelectedItems']),
    async deleteItem() {
      try {
        await this.$http.delete('/errors/', {
          data: {
            ids: [this.error.id],
          },
        });

        this.setSelectedItems(this.getSelectedItems.filter((item) => `${item}` !== this.error.id));
        this.$router.push({ name: 'ErrorHome' });
      } catch (error) {
        this.useToast('Não foi possível excluir o item', 'error');
      }
    },
    async archiveItem() {
      try {
        await this.$http.put('/errors/archive', {
          ids: [this.error.id],
        });

        this.setSelectedItems(this.getSelectedItems.filter((item) => `${item}` !== this.error.id));
        this.$router.push({ name: 'ErrorHome' });
      } catch (error) {
        this.useToast('Não foi possível arquivar o item', 'error');
      }
    },
  },
  created() {
    this.isLoading = true;

    this.$http
      .get(`/errors/${this.id}`)
      .then(({ data }) => {
        this.isLoading = false;
        this.error = data;
      })
      .catch(() => this.$router.push({ name: '404' }));
  },
};
</script>

<style lang="scss" scoped>
@mixin break-large {
  @media (min-width: 1024px) {
    @content;
  }
}

@mixin break-medium-less {
  @media (max-width: 1023px) {
    @content;
  }
}

$color-error: #e84a50;
$color-warning: #eacb1b;
$color-debug: rgba(46, 91, 255, 0.6);

.error-single {
  @include break-medium-less {
    padding: 0 0.5em;
    margin-bottom: 1rem;
  }

  .icon {
    margin-right: 0.25rem;
  }
}
.error-header {
  padding: 1.5rem 0 0.5rem;

  @include break-large {
    margin-bottom: 0;

    .buttons {
      float: right !important;

      .button {
        font-size: 0.75rem;
        border-radius: 2px;

        &:first-child {
          border-bottom-left-radius: 0;
          border-top-left-radius: 0;
        }

        &:not(:last-child) {
          border-bottom-right-radius: 0;
          border-top-right-radius: 0;
        }
      }
    }
  }

  @include break-medium-less {
    .buttons {
      display: flex;

      .button {
        flex-grow: 1;
      }
    }
  }
}
.error-detail {
  background-color: #fff;
  border: 1px solid #e4ebff;
  padding: 1.25rem;

  .error-detail--at {
    color: #8097b1;
  }

  .error-detail--content-date {
    .icon {
      font-size: 1rem;
    }
  }

  @include break-medium-less {
    border-top-width: 4px;

    &.error {
      border-top-color: $color-error;
    }
    &.warning {
      border-top-color: $color-warning;
    }
    &.debug {
      border-top-color: $color-debug;
    }
  }

  .error-detail--content-line {
    margin-top: 2em;

    .small-title-style {
      .icon {
        font-size: 1rem;
      }
    }

    .text {
      padding-left: 1.75rem;
    }
  }
}
.subtitle.date {
  color: #838ea7;
}

.column-message {
  .message {
    background-color: #fff;
    padding-top: 0.75rem;

    .message-body {
      color: #2e384d;
      border-radius: 0;
      border: 0;
      border-left-style: solid;
      padding: 0;

      &.error {
        border-left-color: $color-error;
      }
      &.warning {
        border-left-color: $color-warning;
      }
      &.debug {
        border-left-color: $color-debug;
      }

      @include break-large {
        border-left-width: 4px;
        padding: 0 1.5em 1.25em;
      }

      .tag {
        font-weight: 300;
        padding-left: 2em;
        padding-right: 2em;

        &.error {
          background-color: $color-error;
          color: white;
        }

        &.warning {
          background-color: $color-warning;
          color: white;
        }

        &.debug {
          background-color: $color-debug;
          color: white;
        }
      }

      .small-title-style {
        padding-left: 1.75rem;
      }
    }
  }

  .label-style {
    left: auto;
    top: auto;
    margin-top: 1.6em;
    text-transform: uppercase;
    height: auto;
  }
}
</style>
