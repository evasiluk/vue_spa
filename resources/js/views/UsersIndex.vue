<template>
    <div class="users">
        <div v-if="error" class="error">
            <p>{{ error }}</p>

            <p>
                <button @click.prevent="fetchData">
                    Try Again
                </button>
            </p>
        </div>

        <ul v-if="users">
            <li v-for="{ name, email } in users">
                <strong>Name:</strong> {{ name }},
                <strong>Email:</strong> {{ email }}
            </li>
        </ul>

        <div class="pagination">
            <button :disabled="!prevPage" @click.prevent="goToPrev">Prev</button>
            {{paginationCount}}
            <button :disabled="!nextPage" @click.prevent="goToNext">Next</button>
        </div>
    </div>
</template>
<script>
    import axios from 'axios';
    const getUsers = (page, callback) => {
        const params = {page};

        axios.get("/api/users", {params}).then(response => {
            callback(null, response.data);
        }).catch(error => {
            callback(error, error.response.data);
        })
    };



    export default {
        data() {
            return {
                users: null,
                error: null,
                meta: null,
            }
        },
        beforeRouteEnter (to, from, next) {
            getUsers(to.query.page, (err, data) => {
                next(vm => vm.setData(err, data))
            });
        },
        beforeRouteUpdate (to, from, next) {
            this.users = null;
            getUsers(to.query.page, (err, data) => {
                this.setData(err, data);
                next();
            })
        },
        methods: {
            setData(err, data) {
                if(err) {
                    this.error = err.toString();
                } else {
                    this.users = data.data;
                    this.meta = data.meta;
                }
            },
            goToPrev() {
                this.$router.push({
                    name: 'users.index',
                    query: {
                        page: this.prevPage
                    }
                })
            },
            goToNext() {
                this.$router.push({
                    name: 'users.index',
                    query: {
                        page: this.nextPage
                    }
                })
            },
        },
        computed: {
            paginationCount() {
                if(!this.meta) {
                    return;
                }

                const current_page = this.meta.current_page;
                const last_page = this.meta.last_page;

                return 'Page ' + current_page + ' of ' + last_page;
            },
            nextPage() {
                if(! this.meta || this.meta.current_page === this.meta.last_page) {
                    return;
                }

                return this.meta.current_page + 1;
            },
            prevPage() {
                if(!this.meta || this.meta.current_page === 1) {
                    return;
                }

                return this.meta.current_page - 1;
            }
        }
    }
</script>


