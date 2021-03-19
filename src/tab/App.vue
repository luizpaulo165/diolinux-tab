<template>
    <div wrap-card-page> 
        <div class="loading-container" v-if="loading">
            <span class="pre-image" :style="{'background-image': 'url(../images/diolinux_icon.png)'}"></span>
            <span class="pre-image pulse" :style="{'background-image': 'url(../images/diolinux_icon.png)'}"></span>
            <span class="text">Carregando feed</span>
        </div>
        <transition name="fade">
            <div id="content-page" v-if="!loading">
                <div id="header-page">
                    <h1 id="logo">
                        <img src="../images/diolinux.png" alt="Diolinux" />
                    </h1>
                </div>
                <div 
                    class="wrap-post" 
                    v-for="(item, index) in resource" 
                    :key="'post-' + index" 
                    :active="currentPost == index"
                >
                    <!-- Background Page -->
                    <span 
                        class="bg" 
                        v-if="resourceImg[index]"
                        :style="{'background-image': 'url('+ resourceImg[index]['media_details']['sizes']['thumbnail']['source_url'] +')'}"
                    ></span>
                    
                    <!-- Posts List -->
                    <article class="post-content">
                        <div class="left-cont">
                            <img 
                                v-if="resourceImg[index]"
                                :src="resourceImg[index]['media_details']['sizes']['large']['source_url']" 
                                :alt="item.title.rendered" 
                            />
                        </div>
                        <div class="right-cont">
                            <h2 v-html="item.title.rendered"></h2>
                            <!-- <span class="date">{{ item.date |  time }}</span> -->
                            <p>{{ item.content.rendered | striphtml }}</p>
                            <p><a :href="item.link">Continue lendo</a></p>
                        </div>
                    </article>
                </div>

                <!-- Pagination -->
                <div 
                    id="pagination"
                    @mouseenter="stopAutoplay"
                    @mouseleave="startAutoplay"
                >
                    <button 
                        v-for="(page, index) in resource" 
                        :key="'page' + index" 
                        :active="currentPost == index" 
                        @click="nextPost(index)"
                    >
                        <span></span>
                    </button>
                </div>

                <!-- Footer -->
                <div id="footer-page">
                    <address>
                        <i>© Blog <a href="https://diolinux.com.br/">Diolinux</a> © all rights reserved</i>
                    </address>
                    <span>Extenssion developed by <a href="https://twitter.com/papaulov">@papaulov</a>.</span>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
import axios from 'axios';
import moment from 'moment-timezone';

export default {
    filters: {
        striphtml(value) {
            let div = document.createElement("div");
            div.innerHTML = value;

            let text = div.textContent || div.innerText || "";

            return text.substring(0,235) + '...';
        },
        time(value) {
            let dateThis = moment(value);

            return dateThis.format('DD/MM/YY');
        },
        titleCharacters(value) {
            return value.replace(/&#8211;/g, '&ndash;')
        }
    },
    computed: {
        resourceFilter() {
            let arr = this.resource;

            return arr;
        }
    },
    data () {
        return {
            loading: true,
            errored: false,

            api_link: 'https://diolinux.com.br/wp-json/wp/v2/posts',
            resource: [],
            resourceImg: [],

            currentPost: 0,
            timerMax: 6000,
            timerCounter: null
        }
    },
    methods: {
        loadingResource() {
            const self = this;
            // Loading API content
            axios
                .get(self.api_link)
                .then(response => {
                    let API = response.data;

                    // add content this.resource
                    self.resource = response.data;

                    // add images about contents
                    API.filter(cur => {
                        let urlLinks = cur._links['wp:featuredmedia'][0].href;

                        axios
                        .get(urlLinks)
                        .then(res => {
                            self.resourceImg.push(res.data);
                        })
                        .catch(error => {
                            // console.log(error)
                            self.errored = true;
                        })
                    })
                })
                .catch(error => {
                    console.log(error)
                    self.errored = true;
                })
                .finally(() => {
                    self.loading = false
                    this.startAutoplay();
                });
        },
        nextPost(value) {
            this.currentPost = value;
        },
        startAutoplay() {
            this.timerCounter = setInterval(() => {
                if (this.currentPost == this.resource.length - 1){
                    this.currentPost = 0;
                } else {
                    this.currentPost += 1;
                }
            }, this.timerMax);
        },
        stopAutoplay() {
            clearInterval(this.timerCounter);
        }
    },
    mounted () {
        this.loadingResource();
    },
}
</script>

<style scoped lang="scss">
    @import 'tab.css';

    .loading-container{
        text-align: center;
        position:absolute;
        top:50%;
        left:50%;
        transform:translate(-50%,-50%);

        .pre-image{
            background-position:left center;
            background-size:106px;
            background-repeat: no-repeat;
            width:106px;
            height:106px;
            height:0;
            display:block;
            padding-top:100%;
            position:relative;
            z-index: 1;
        }

        .pre-image.pulse{
            filter:blur(42px) contrast(5);
            width:106px;
            position:absolute;
            top:0;
            left:0;
            z-index:0;
            animation: pulse 1.8s infinite;
        }
        @keyframes pulse {
            0% {opacity:0;}
            50% {opacity:0.8;}
            100% {opacity:0.1;}
        }

        .text{
            text-align: center;
            text-transform:uppercase;
            font-size:0.6vw;
            color:#fff;
            display:block;
            padding-top:0.5rem;
        }
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
    }
    .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
        opacity: 0;
    }

    #pagination{
        // background:rgba(0,0,0,0.6);
        width:10px;
        display:flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        position:fixed;
        // padding:1rem 2rem;
        // border-radius:6px 0 0 6px;
        top:50%;
        right:2rem;
        z-index:20;
        transform:translateY(-50%);
        box-sizing: border-box;

        button{
            background:transparent;
            font-size:0;
            width:20px;
            height:20px;
            padding:0;
            border:0;
            position: relative;
            opacity:0.4;
            transition:0.1s all linear;
            cursor:pointer;

            span{
                background-color:#fff;
                background-position:center;
                background-size:cover;
                background-repeat: no-repeat;
                width:10px;
                height:10px;
                display:block;
                border-radius:50px;
                overflow: hidden;
                position:relative;
                z-index:1;
            }

            &:before{
                filter:blur(15px) contrast(10);
                background:#ec00ff;
                content:'';
                width:8px;
                height:8px;
                display:block;
                border-radius:50px;
                position: absolute;
                top:50%;
                right:-3rem;
                transform:translateY(-50%);
                opacity:0;
                transition:0.1s all linear;
                z-index:0;
            }
            &:hover{
                opacity:1;
            }
            &:last-child{
                margin-bottom:0;
            }
            &[active]{
                pointer-events: none;
                opacity:1;
                transition:0.2s all linear;

                &:before{
                    right:-1.5rem;
                    opacity:1;
                    transition:0.2s all linear;
                }
            }
        }
    }

    .wrap-post{
        width:100vw;
        height:100vh;
        display:flex;
        align-items:center;
        justify-content: center;
        flex-direction: row;
        box-sizing: border-box;
        position:absolute;
        opacity:0;
        z-index:0;
        transition:0.2s all linear;

        &[active]{
            top:0;
            left:0;
            opacity:1;
            z-index:10;
            transition-delay:1s;
            transition:1s all linear;
        }
        &:before{
            background: rgb(0,0,0);
            background: radial-gradient(circle, rgba(0,0,0,1) 0%, rgba(2,0,36,0) 100%);
            content:'';
            width:inherit;
            height:inherit;
            display:block;
            position:absolute;
            left:0;
            bottom:0;
            z-index:1;
            /* opacity:0.5; */
        }
        .bg{
            filter:blur(15px) contrast(120%);
            background-size:cover;
            background-repeat: no-repeat;
            background-position: center;
            width:110%;
            height:110%;
            display:block;
            position:absolute;
            left:50%;
            top:50%;
            z-index:0;
            opacity:0.8;
            transform:translate(-50%,-50%);
        }
    }
    

    .post-content{
        flex:1;
        max-width:calc(100% - 3rem);
        display:flex;
        align-items: center;
        justify-content: center;
        flex-direction: row;
        padding:20px;
        position:relative;
        z-index:10;
        box-sizing: border-box;

        .left-cont{
            text-align: right;
            flex:0 0 40%;

            img{
                width:100%;
                border-radius:10px;
            }
        }
        .right-cont{
            flex:0 0 40%;
            padding:0 1.5rem;

            h2{
                font-size: 1.8vw;
                line-height:2vw;
                font-weight: 700;
                color:#fff;
                margin-bottom:15px;
            }
            p{
                font-size: 1vw;
                line-height:1.6vw;
                color:#fff;
                margin-bottom:20px;
                
                a{
                    background: rgb(1,105,255);
                    background: linear-gradient(135deg, rgba(1,105,255,1) 45%, rgba(236,0,255,1) 100%); 
                    font-size:0.8vw;
                    color:#fff;
                    display:inline-block;
                    padding:6px 14px 8px;
                    border-radius:4px;
                    text-decoration: none;
                    transition:0.1s all linear;
                }
                &:last-child{
                    margin-bottom:0;
                }
            }
        }
    }

    #header-page{
        width:100%;
        height:48px;
        position: fixed;
        top:0;
        left:0;
        z-index:100;
    }

    #footer-page{
        width:100%;
        height:32px;
        display:flex;
        align-items: center;
        justify-content: center;
        padding:0 2rem;
        position:fixed;
        bottom:0;
        left:0;
        z-index:20;
        box-sizing: border-box;

        address,
        span{
            font-style: normal;
            font-size: 0.6vw;
            color: rgba(255,255,255,0.5);
        }
        a{
            color: rgba(255,255,255,0.5);
        }
        address{
            text-align:center;
            flex:1;
        }
        span{
            display:block;
            position: absolute;
            top:50%;
            right:2rem;
            transform:translateY(-50%);
        }
    }
</style>