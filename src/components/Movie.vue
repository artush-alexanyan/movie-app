<template>
    <div class="container">
    <h4 class="text-center mt-5">Favorites</h4>    
    <div v-if="favorites.length > 0">
        <table class="table mt-5">
        <thead>
            <tr>
            <th scope="col">No:</th>
            <th scope="col">imdbID</th>
            <th scope="col">Title</th>
            <th scope="col">Year</th>
            </tr>
        </thead>
        <tbody> 
            <tr v-for="(f, i) in favorites" :key="f.title">
            <th> {{ i + 1 }} </th>
            <th scope="row"> {{ f.imdbID }} </th>
            <td> {{ f.Title }} </td>
            <td> {{ f.Year }} </td>
            <td @click="removeFavorites(i)"> 
                <i class="far fa-star yellow"></i> 
            </td>
            </tr>
        </tbody>
        </table>                                  
    </div>
    <div v-else>
        <h5 class="text-center text-secondary mt-5"> No favorites added </h5>
    </div>


    <div v-if="loading" class="mt-5 mb-5">
        <Loader />
    </div>

    <div class="container mt-5">
        <div class="row" v-if="!loading">
        <div class="col-2 mt-3" v-for="(movie, i) in movies" :key="i">
            <div class="card">
            <div class="card-body">
                <h6 class="card-title"> 
                    <strong>
                        {{ movie.Title }}
                    </strong> 
                </h6>
                <p class="card-text"> {{ movie.Year }} </p>
                <strong @click="addToFavorites(i)">
                    <i class="far fa-star"></i>
                </strong> 
            </div>
            </div>
        </div>
        </div>         
    </div>            
    <div class="d-flex justify-content-center">
        <nav aria-label="..." class="mt-3">
        <ul class="pagination">
            <li class="page-item" 
                :class="[ page > 0 ? 'active' : '' ]"
                >
                <span class="page-link">Previous</span>
            </li>

            <li class="page-item">
                <span class="page-link" 
                      v-show="page > 10">
                      ...
                </span>
            </li>

            <li class="page-item" 
                :class="[ page == i ? 'active' : '' ]"
                v-for="(p, i) in getPages" 
                :key="i"
                @click="getMovie(i)"
                >
                <a class="page-link"> {{ p }} </a>
            </li>

            <li class="page-item">
                <span class="page-link" 
                      v-show="page < 10">
                      ...
                </span>
            </li>            

            <li class="page-item" 
                :class="[ page < getPages - 1 ? 'active' : '' ]" 
                @click="onNext"
            >
                <span class="page-link">Next</span>
            </li>
        </ul>
        </nav>          
    </div>              
    </div>
</template>

<script>
import Loader from '../components/Loader'
import api from '../api'

export default {
    name: 'Movie',
    components: { Loader },
    data: () => ({
        movies: [],
        favorites: [],
        page: 0,
        per_page: null,
        total: null,
        total_pages: null,
        loading: false,
    }),
    computed: {
        getPages () {
            return this.movies.length > 0 
                   ? Math.round(this.total_pages / this.per_page)
                   : 0
        },
    },
    methods: {
        getMovie (i = 0) {
            this.loading = true
            this.page = i
            api.get(`movies/search/?Title=&page=${ this.page }`)
            .then(response => {
                const data = response.data
                this.movies = data.data
                this.total = data.total
                this.per_page = data.per_page
                this.total_pages = data.total_pages
                this.loading = false
            })
            .catch(err => {
                console.log("Error", err)
            })
        },
        onNext () {
            if(this.page < this.getPages - 1) {
                this.loading = true
                this.page++
                api.get(`movies/search/?Title=&page=${ this.page }`)
                .then(response => {
                    const data = response.data
                    this.movies = data.data
                    this.total = data.total
                    this.per_page = data.per_page
                    this.total_pages = data.total_pages
                    this.loading = false
                })
                .catch(err => {
                    console.log("Error", err)
                })                   
            }
        },
        addToFavorites (i) {
            this.movies.filter((item, index) => {
                i == index
                if(i == index && !this.favorites.includes(item)){
                    this.favorites.push(item)
                }
            })
        },
        removeFavorites (i) {
            this.favorites.splice(i, 1)
        }
    },
    created () {
        this.getMovie()
    }
}
</script>

<style scoped>
li,i {
    cursor: pointer;
}
.yellow{
    color: orange;
}
.current{
    background: red;
}
</style>