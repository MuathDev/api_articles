<template>
  <div dir="rtl" style="text-align:center;">
    <h2>المقالات</h2>
    
    <form @submit.prevent="addArticle" class="mb-3">
      <div class="form-group">
        <input type="text" class="form-control" placeholder="Title" v-model="article.title">
      </div>
      <div class="form-group">
        <textarea class="form-control" placeholder="Body" v-model="article.body"></textarea>
      </div>
      <button type="submit" class="btn btn-light btn-block">حفظ</button>
    </form>
    <button @click="clearForm()" class="btn btn-danger btn-block">إلغاء</button>


    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">السابق</a></li>

        <li class="page-item disabled"><a class="page-link text-dark" href="#">صفحة {{ pagination.current_page }} من {{ pagination.last_page }}</a></li>
    
        <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">التالي</a></li>
      </ul>
    </nav>


    <div class="card card-body mb-2" v-for="article in articles" v-bind:key="article.id">
      <h3>{{ article.title }}</h3>
      <p>{{ article.body }}</p>
      <hr>
      <button @click="editArticle(article)" class="btn btn-warning mb-2">تعديل</button>
      <button @click="deleteArticle(article.id)" class="btn btn-danger">حذف</button>
    </div>
  </div>

</template>


<script>

export default {
  data() {
    return {
      articles: [],
      article: {
        id: '',
        title: '',
        body: ''
      },
      article_id: '',
      pagination: {},
      edit: false
    };
  },

  created() {
    this.fetchArticles();
  },

  methods: {


    fetchArticles(page_url) {
      let vm = this;
      page_url = page_url || 'api/articles';
      fetch(page_url)
        .then(res => res.json())
        .then(res => {
          this.articles = res.data;
          vm.makePagination(res.meta, res.links);
        })
        .catch(err => console.log(err));
    }
    
    ,makePagination(meta, links) {
      let pagination = {
        current_page: meta.current_page,
        last_page: meta.last_page,
        next_page_url: links.next,
        prev_page_url: links.prev
      };

      this.pagination = pagination;
    }
    
    ,deleteArticle(id) {
      if (confirm('هل انت متأكد من عملية الحذف؟')) {
        fetch(`api/article/${id}`, {
          method: 'delete'
        })
          .then(res => res.json())
          .then(data => {
            alert('تمت الحذف بنجاح');
            this.fetchArticles();
          })
          .catch(err => console.log(err));
      }
    }

    ,addArticle() {
      if (this.edit === false) {
        // Add
        fetch('api/article', {
          method: 'post',
          body: JSON.stringify(this.article),
          headers: {
            'content-type': 'application/json'
          }
        })
          .then(res => res.json())
          .then(data => {
            this.clearForm();
            alert('تمت الاضافة بنجاح');
            this.fetchArticles();
          })
          .catch(err => console.log(err));
      } else {
        // Update
        fetch('api/article', {
          method: 'put',
          body: JSON.stringify(this.article),
          headers: {
            'content-type': 'application/json'
          }
        })
          .then(res => res.json())
          .then(data => {
            this.clearForm();
            alert('تمت التحديث بنجاح');
            this.fetchArticles();
          })
          .catch(err => console.log(err));
      }
    }

    ,editArticle(article) {
      this.edit = true;
      this.article.id = article.id;
      this.article.article_id = article.id;
      this.article.title = article.title;
      this.article.body = article.body;
    }

    ,clearForm() {
      this.edit = false;
      this.article.id = null;
      this.article.article_id = null;
      this.article.title = '';
      this.article.body = '';
    }

  }
};
</script>
