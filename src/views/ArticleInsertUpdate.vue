<template>
  <article>
    <header>
      <h2 class="title">{{ headerTitle }} Artigo</h2>
    </header>

    <div class="columns is-centered content-edit">
      <div class="box-edit">
        <!-- response error -->
        <div v-if="error">
          <div class="notification is-danger">
            {{ error }}
          </div>
        </div>

        <!-- title -->
        <label class="label-login" for="titleInput"> Título </label>
        <input
          type="text"
          id="titleInput"
          class="input"
          placeholder="Título"
          maxlength="100"
          v-model="title"
        />

        <!-- description -->
        <label class="label-login" for="descriptionInput"> Descrição </label>
        <input
          type="text"
          id="descriptionInput"
          class="input"
          placeholder="Descrição"
          maxlength="250"
          v-model="description"
        />

        <!-- text -->
        <label class="label-login" for="textTextArea"> Texto </label>
        <textarea
          id="textTextArea"
          cols="30"
          rows="10"
          class="input textArticleInput"
          placeholder="Texto"
          v-model="text"
        ></textarea>

        <!-- category -->
        <div class="columns">
        <div class="column is-half">
          <label class="label-login" for="categorySelect"> Categoria </label>
          <select id="categorySelect" class="input" v-model="category">
            <option value="0">Escolha uma opção</option>
            <option v-for="c in categories" :key="c.id" v-bind:value="c.id">
              {{ c.description }}
            </option>
          </select>
        </div>

        <!-- subject -->
        
          <div class="column is-half">
            <label class="label-login" for="subjectSelect"> Matéria </label>
            <select id="subjectSelect" class="input" v-model="subject">
              <option value="0">Escolha uma opção</option>
              <option v-for="s in subjects" :key="s.id" v-bind:value="s.id">
                {{ s.description }}
              </option>
            </select>
          </div>
        </div>

        <hr />

        <div class="btns">
          <button type="button" class="button is-outlined" @click="getBack()">
            Voltar
          </button>

          <button type="button" class="button is-success" @click="save()">
            Salvar
          </button>
        </div>
      </div>
    </div>
  </article>
</template>

<script>
import Filters from "../js/others/Filters";
import { getSessionUser } from "../js/others/Helpers";

import ArticleServices from "../js/services/ArticleServices";
import CategoryServices from "../js/services/CategoryServices";
import SubjectServices from "../js/services/SubjectServices";

export default {
  created() {
    const lastPath = this.$route.params.lastPath;
    if (lastPath) {
      this.lastPath = lastPath;
    }

    const currentId = this.$router?.currentRoute?.params?.id || undefined;
    this.headerTitle = currentId ? "Atualizar" : "Novo";
    if (currentId) {
      this.id = currentId;
      ArticleServices.getById(currentId)
        .then((response) => {
          console.log("response get article", response);
          const article = response.data;
          if (article && article.id) {
            this.author = article.author_id;
            this.category = article.category_id;
            this.description = article.description;
            this.last_changed = article.last_changed;
            this.published_date = article.published_date;
            this.status_article = article.status_article;
            this.subject = article.subject_id;
            this.text = article.text;
            this.title = article.title;
          }
        })
        .catch((error) => {
          console.log("error get article", error);
          this.$router.push({ name: "Articles" });
        });
    }

    CategoryServices.getCategories()
      .then((response) => {
        this.categories = response.data;
      })
      .catch((error) => {
        alert("error", error);
        this.categories = [];
      });

    SubjectServices.getSubjects()
      .then((response) => (this.subjects = response.data))
      .catch((error) => {
        alert("error", error);
        this.subjects = [];
      });
  },

  data() {
    return {
      id: -1,
      description: "",
      last_changed: "",
      published_date: "",
      status_article: "",
      subject: 0,
      text: "",
      title: "",
      headerTitle: "",
      author: 0,
      category: 0,
      error: undefined,
      categories: [],
      subjects: [],
      lastPath: "Articles",
    };
  },

  methods: {
    validation() {
      if (this.title === '' || this.title.length === 0) {
        alert('Informe o título!');
        return false;
      }

      if (this.description === '' || this.description.length === 0) {
        alert('Informe a descrição!');
        return false;
      }

      if (this.text === '' || this.text.length === 0) {
        alert('Informe o texto!');
        return false;
      }

      if (this.category === '' || this.category === 0 || this.category === '0') {
        alert('Informe a categoria!');
        return false;
      }

      if (this.subject === '' || this.subject === 0 || this.subject === '0') {
        alert('Informe a matéria!');
        return false;
      }

      return true;
    },


    save() {
      if (this.validation()) {
        const payload = {
          id: this.id,
          description: this.description,
          status_article: this.status_article,
          subject: this.subject,
          text: this.text,
          title: this.title,
          author: this.author,
          category: this.category,
        };

        console.log("payload", payload);

        if (this.id === -1) {
          payload.author = getSessionUser().id;
          this.insert(payload);
        } else {
          payload.status_article = 0;
          this.update(payload);
        }
      }
    },

    insert(payload) {
      ArticleServices.insert(payload)
        .then((response) => {
          console.log("ArticleServices insert response", response);
          if (response.data && response.data !== "") {
            alert("Novo artigo criado com sucesso!");
            location.href = `/Articles?id=${payload.author}`;
          } else {
            alert("Erro ao inserir o novo artigo!");
          }
        })
        .catch((error) => {
          console.log("insert error", error);
          alert("Erro ao inserir o novo artigo!");
        });
    },

    update(payload) {
      ArticleServices.update(payload)
        .then((response) => {
          console.log("ArticleServices update response", response);
          if (response.data && response.data !== "") {
            alert("Artigo atualizado com sucesso!");
            location.href = `/Articles?id=${payload.author}`;
          } else {
            alert("Erro ao inserir o novo artigo!");
          }
        })
        .catch((error) => {
          console.log("update error", error);
          alert("Erro ao atualizar o artigo!");
        });
    },

    getBack() {
      window.location.href = this.lastPath;
    },
  },

  filters: {
    formatDatetime: Filters.formatDatetime,
  },
};
</script>

<style scoped>
.title {
  background: #713a3a;
  color: #ffffff;
  padding: 10px;
  font-size: 20px;
  margin-bottom: 0;
}

.label-login {
  float: left;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  font-size: 14px;
  font-weight: 600;
  margin: 15px 0 5px 0;
}

input {
  margin-bottom: 10px;
}

.btns {
  display: flex;
  justify-content: space-between;
}

.box-edit {
  padding: 25px;
  background: #dddddd;
  margin-bottom: 0;
  width: 80%;
}

.box-edit hr {
  background-color: #cccccc;
  border: none;
  display: block;
  height: 2px;
}

.foto-perfil {
  background: #375269;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 0 !important;
}

.content-edit {
  margin-top: 10px;
}

.textArticleInput {
  min-height: 300px;
}
</style>