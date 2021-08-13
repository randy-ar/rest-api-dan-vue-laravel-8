<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <div class="card mb-3" v-for="post in posts" v-bind:key="post.id">
                    <div class="card-header">
                        {{ post.title.slice(0, 80) }}...
                    </div>

                    <div class="card-body" style="text-align: justify;">
                        <small class="text-muted d-block mb-1">{{ post.created_at | formatDate }}</small>
                        {{ post.body.slice(0, 200) }}... Read more
                    </div>
                    <div class="card-footer border-0 bg-white">
                        <div class="d-flex justify-content-end">
                            <button class="btn btn-success me-3" @click="editPost(post)">Edit</button>
                            <button class="btn btn-danger" @click="deletePost(post.id)">Delete</button>
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card">
                    <div class="card-header">
                        <div class="card-title" id="form-title">
                            Buat Post
                        </div>
                    </div>
                    <form @submit.prevent="createPost()">
                    <div class="card-body">
                        <div class="mb-3">
                            <label for="title">Judul Post</label>
                            <input type="text" class="form-control" v-model="post.title">
                        </div>
                        <div class="mb-3">
                            <label for="body">Isi Post</label>
                            <textarea rows="10" class="form-control" v-model="post.body"></textarea>
                        </div>
                    </div>
                    <div class="card-footer bg-white border-0">
                        <button class="btn btn-primary" id="button-add" type="submit">Simpan</button>
                    </div>
                    </form>
                </div>
            </div>
            <div class="col-md-12">
                <nav aria-label="Page navigation example">
                    <ul class="pagination">
                        <li class="page-item" v-bind:class="[{disabled : !pagination.prev_page}]"><a class="page-link"  href="#" @click="fetchPost(pagination.prev_page)">Previous</a></li>
                        <li class="page-item disabled"><a class="page-link text-dark" href="#">Page {{ pagination.current_page }} of {{ pagination.last_page }}</a></li>
                        <li class="page-item" v-bind:class="[{disabled : !pagination.next_page}]"><a class="page-link" href="#" @click="fetchPost(pagination.next_page)">Next</a></li>
                    </ul>
                </nav>
            </div>
        </div>
    </div>
    
</template>

<script>
export default {
    data() {
        return {
            posts : [],
            post_id : '',
            post : {
                id : '',
                title : '',
                body : '',
            },
            pagination : {},
            edit : false,
        }
    },
    created() {
        this.fetchPost();
    },
    methods : {
        fetchPost(page_url){
            page_url = page_url || 'api/post';
            let this_nuts = this;
            fetch(page_url)
                .then(res => res.json())
                .then(res => {
                    this.posts = res.data;
                    this_nuts.makePagination(res.meta, res.links);
                }).catch(err => console.log(err));
        },
        makePagination(meta, links){
            let pagination = {
                current_page : meta.current_page,
                last_page : meta.last_page,
                next_page : links.next,
                prev_page : links.prev,
            }
            this.pagination = pagination;
            console.log(this.pagination);
        },
        deletePost(id){
            if(confirm('Hapus post ini ?')){
                fetch(`api/post/delete/${id}`, {
                    method: 'delete'
                })
                .then(res => res.json())
                .then(res => {
                    alert(`Post telah telah dihapus`);
                    this.fetchPost();
                }).catch(err => console.log(err));
            }
        },
        createPost(){
            if(this.edit === false){
                fetch('api/post/store', {
                    method : 'post',
                    body : JSON.stringify(this.post),
                    headers: {
                        'content-type' : 'application/json'
                    }
                })
                .then(res => res.json())
                .then(res => {
                    this.clearPost();
                    alert('Post telah ditambah');
                    this.fetchPost();
                }).catch(err => console.log(err));
            }else{
                console.log(this.post);
                fetch(`api/post/update/${this.post.id}`, {
                    method : 'put',
                    body : JSON.stringify(this.post),
                    headers: {
                        'content-type' : 'application/json'
                    }
                })
                .then(res => res.json())
                .then(res => {
                    alert('Post telah diupdate');
                    this.clearPost();
                    this.fetchPost();
                }).catch(err => console.log(err));
            }
        },
        clearPost(){
            this.post.title = '';
            this.post.body = '';
            this.post_id = null;
            this.post.id = null;
            this.edit = false;
        },
        editPost(post){
            this.edit = true;
            this.post.id = post.id;
            this.post_id = post.id;
            this.post.title = post.title;
            this.post.body = post.body;
        }
    }
}
</script>