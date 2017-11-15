<script>
import tantan from '../../tantan';

const nextTick = () => new Promise(( resolve ) => process.nextTick(resolve));

export default {
    name: 'app',
    components: {
        tantan,
    },
    data () {
        return {
            // data
            list: [
                {
                    avatar: 'https://gss1.bdstatic.com/9vo3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=abef1d2dae0f4bfb8cd099523b741fcd/2fdda3cc7cd98d1028047e832b3fb80e7aec90ab.jpg',
                    username: 'Emma Watson',
                },
                {
                    avatar: 'https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/s%3D220/sign=58c6e244b63eb13540c7b0b9961fa8cb/38dbb6fd5266d016d1e850849d2bd40734fa35ad.jpg',
                    username: 'Emma Stone',
                },
                {
                    avatar: 'https://gss1.bdstatic.com/9vo3dSag_xI4khGkpoWK1HF6hhy/baike/s%3D220/sign=7073e30ecf177f3e1434fb0f40ce3bb9/43a7d933c895d1430cf0959b75f082025baf0787.jpg',
                    username: 'Jennifer Lawrence',
                },
                {
                    avatar: 'https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=a3504c7c104c510faec4e51c58624210/7c1ed21b0ef41bd5897a7a405bda81cb39db3d5c.jpg',
                    username: 'Emilia Clarke',
                },
            ],
            // progress @extend tantan.progress
            progress: 0,
            // draging status flag
            draging: false,
            // random rotate direction
            direction: 1,
        };
    },
    created () {
        window.app = this;
    },
    methods: {
        touchstart ({ event, el }) {
            // console.log('touchstart', event, el);
        },
        touchmove ({ event, el }) {
            // console.log('touchmove', event, el);
        },
        touchend ({ event, el }) {
            // console.log('touchend', event, el);
        },
        touchcancel ({ event, el }) {
            // console.log('touchcancel', event, el);
        },
        transform ({ event, el, progress, matrix }) {
            const { type } = event;
            if (type == 'touchmove') {
                // handler touchmove transform
                // make a rotation
                const angle = 1.5 * this.direction;
                const el = this.$refs.matrix;
                el.style.cssText = `transform:rotate(${ angle * Math.min(1, Math.max(0, Math.abs(progress))) * ((+(progress >= 0)) * 2 - 1) }deg)`;
                const [ a, b, c, d, e, f ] = getComputedStyle(el).transform.slice(7, -1).replace(/\s/g, '').split(',');
                const rotation = [a, b, c, d];
                matrix(([ a, b, c, d, e, f ]) => {
                    return rotation.concat(e, f);
                });
            }
            if (type == 'touchend' || type == 'touchcancel') {
                // handler touchend transform
                if (Math.abs(progress) >= 1) {
                    // slideleft
                    // slideright
                    // random to change direction
                    const random = Math.random() - 0.5;
                    this.direction *= (+(random >= 0)) * 2 - 1;
                    // handler slide out transform
                    // make an inertia in vertical axis
                    const [ a, b, c, d, e, f ] = getComputedStyle(el).transform.slice(7, -1).replace(/\s/g, '').split(',');
                    const destination = [a, b, c, d, e, f];
                    matrix(([ a, b, c, d, e, f ]) => {
                        destination[4] = e;
                        return destination;
                    });
                } else {
                    // slideback
                    // slide back the original position as default
                }
            }
        },
        drag ({ event, el, progress }) {
            // console.log('drag', progress);
            this.draging = true;
            this.progress = progress;
        },
        drop ({ event, el, progress }) {
            // console.log('drop', progress);
            this.draging = false;
            this.progress = 0;
        },
        slideleft ({ event, el }) {
            // console.log('slideleft');
        },
        slideright ({ event, el }) {
            // console.log('slideright');
        },
        slideback ({ event, el }) {
            // console.log('slideback');
        },
        slidebackend ({ event, el }) {
            // console.log('slidebackend');
        },
        async slideleftend ({ event, el }) {
            // console.log('slideleftend');
            return;
            const data = this.list.shift();
            console.log(`forget ${ data.username }`);
            await nextTick();
            this.list.push(data);
        },
        async sliderightend ({ event, el }) {
            // console.log('sliderightend');
            return;
            const data = this.list.shift();
            console.log(`like ${ data.username }`);
            await nextTick();
            this.list.push(data);
        },
    },
};
</script>

<template>
    <div id="app">
        <div class="main">
            <div class="container">
                <tantan
                    ref="tantan"
                    :list="list"
                    @touchstart="touchstart"
                    @touchmove="touchmove"
                    @touchend="touchend"
                    @touchcancel="touchcancel"
                    @transform="transform"
                    @drag="drag"
                    @drop="drop"
                    @slideback="slideback"
                    @slideleft="slideleft"
                    @slideright="slideright"
                    @slidebackend="slidebackend"
                    @slideleftend="slideleftend"
                    @sliderightend="sliderightend"
                >
                    <!-- use "slot-scope" in Vue 2.5.0+ -->
                    <!-- use "scope" in Vue 2.4.4- -->
                    <template slot="card" slot-scope="props">
                        <div class="card" :index="props.index" :class="{ 'draging': draging }">
                            <div class="card-avatar" :style="{ 'background-image': `url(${ props.item.avatar })` }"></div>
                            <div class="card-username">{{ props.item.username }}</div>
                            <div class="card-forget"
                                v-if="props.index == 0"
                                :class="{ 'draging': draging }"
                                :style="{ 'opacity': Math.min(1, Math.max(0, -progress)) }"
                            ><i>×</i></div>
                            <div class="card-like"
                            v-if="props.index == 0"
                                :class="{ 'draging': draging }"
                                :style="{ 'opacity': Math.min(1, Math.max(0, progress)) }"
                            ><i>❤</i></div>
                        </div>
                    </template>
                </tantan>
            </div>
        </div>
        <div class="bottom">
            <div class="forget">
                <i :class="{ 'draging': draging }"
                    :style="{ 'transform': `scale(${ 1 + Math.min(1, Math.max(0, -progress)) * 0.15 })` }"
                >×</i>
            </div>
            <div class="like">
                <i :class="{ 'draging': draging }"
                    :style="{ 'transform': `scale(${ 1 + Math.min(1, Math.max(0, progress)) * 0.15 })` }"
                >❤</i>
            </div>
        </div>
        <div class="matrix" ref="matrix"></div>
    </div>
</template>

<style>
    /* basically layout */
    html,
    body {
        margin: 0;
        padding: 0;
        background-color: #f6f5f1;
    }
    #app {
        overflow: hidden;
        width: 100%;
    }
    .main {
        width: 100%;
        /* 814 / 640 ≈ 1.27 */
        padding-bottom: 127%;
        position: relative;
        margin: 10px 0 30px;
    }
    .container {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        box-sizing: border-box;
        padding: 0 10px;
        z-index: 2;
    }
    .bottom {
        font-size: 0;
        text-align: center;
        position: relative;
        z-index: 1;
    }
</style>

<style>
    /* each card style */
    .card {
        height: 100%;
        border-radius: 6px;
        box-sizing: border-box;
        border: .5px solid #e8e4e4;
        background-color: #ffffff;
        position: relative;
    }
    .card-avatar {
        width: 100%;
        /* 614 / 594 ≈ 1.03 */
        padding-bottom: 103%;
        background-size: cover;
        background-repeat: no-repeat;
        background-position: center center;
    }
    .card-username {
        padding: 0 15px;
        overflow: hidden;
        white-space: nowrap;
        word-wrap: break-word;
        word-break: break-all;
        text-overflow: ellipsis;
        font-size: 20px;
        height: 24px;
        line-height: 24px;
        margin-top: 10px;
    }
    .card-forget,
    .card-like {
        position: absolute;
        top: 20px;
        width: 85px;
        height: 85px;
        border-radius: 50%;
        opacity: 0;
        font-size: 0;
    }
    .card-forget {
        right: 15px;
        background-color: #d3d2cc;
    }
    .card-like {
        left: 15px;
        background-color: #ec553f;
    }
    .card-forget i,
    .card-like i {
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        width: 54px;
        height: 54px;
        margin: auto;
        display: block;
        position: absolute;
        border-radius: 50%;
        background-repeat: no-repeat;
        background-position: center center;
    }
    .card-forget i {
        background-size: 29.6px 29.6px;
        background-image: url('assets/forget-white.png');
    }
    .card-like i {
        background-size: 41px 36px;
        background-image: url('assets/like-white.png');
    }
</style>

<style>
    /* button style */
    .forget,
    .like {
        width: 70px;
        height: 70px;
        position: relative;
        display: inline-block;
        border-radius: 50%;
        background-color: #efede8;
        margin: 0 6px;
    }
    .forget i,
    .like i {
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        width: 54px;
        height: 54px;
        margin: auto;
        display: block;
        position: absolute;
        border-radius: 50%;
        background-color: #fff;
        background-repeat: no-repeat;
        background-position: center center;
    }
    .forget i {
        background-size: 18.5px 18.5px;
        background-image: url('assets/forget.png');
    }
    .like i {
        background-size: 25.5px 22.5px;
        background-image: url('assets/like.png');
    }
</style>

<style>
    /* transition */
    .card,
    .like i,
    .forget i,
    .card-like,
    .card-forget {
        transition: all 2.15s;
    }
    .card.draging,
    .like i.draging,
    .forget i.draging,
    .card-like.draging,
    .card-forget.draging {
        transition: none;
    }
</style>

<style>
    /* card pack style */
    .card {
        transform: scale(.91);
        transform-origin: 50% 140%;
    }
    .card[index="0"] {
        transform: scale(1);
    }
    .card[index="1"] {
        transform: scale(.97);
    }
    .card[index="2"] {
        transform: scale(.94);
    }
</style>

<style>
    /* use matrix element to calc matrix */
    .matrix {
        opacity: 0;
        position: absolute;
        pointer-events: none;
    }
</style>
