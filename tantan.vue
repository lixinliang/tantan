<script>

// cssText helper
function css ( obj ) {
    return Object.keys(obj).map(( key ) => {
        const value = obj[key];
        return `${ key }:${ (value instanceof Array ? value.join('') : value) }`;
    }).join(';');
}

// css value function helper
css.fn = function ( name, args ) {
    return `${ name }(${ args.join(',') })`;
};

// slice to array
function slice ( object ) {
    return [].slice.call(object);
}

export default {
    props: {
        // transition duration of slide back
        bounceSpeed: {
            type: Number,
            default: 250,
        },
        // transition timing function of slide back
        bounceTimingFunction: {
            type: String,
            default: css.fn('cubic-bezier', [0, 0, 0.59, 1.12]),
        },
        // transition duration of slide out
        escapeSpeed: {
            type: Number,
            default: 300,
        },
        // transition timing function of slide out
        escapeTimingFunction: {
            type: String,
            default: css.fn('cubic-bezier', [0, 0, 0.59, 1.12]),
        },
        // threshold of slide out
        escapeThreshold: {
            type: Number,
            default: 60,
        },
        // disable touch event
        disabled: {
            type: Boolean,
            default: false,
        },
        // card pack list
        list: {
            type: Array,
            default: [],
        },
    },
    computed : {
        data () {
            return this.list.slice().reverse();
        },
        length () {
            return this.data.length;
        },
    },
    data () {
        return {
            // save prev touch point
            prevpoint: null,
            // save touch start point
            startpoint: null,
            // save offset x / y between touch start point and origin touch start point when item is transitioning
            originpoint: null,
            // progress with slide action
            progress: 0,
        };
    },
    created () {
        this.$on('tantan:slideleft', () => {
            const { progress } = this;
            const duration = this.escapeSpeed;
            const bezier = this.escapeTimingFunction;
            const pointerEvents = 'none';
            const { items } = this.$refs;
            const el = items[items.length - 1];
            this.$emit('slideleft');
            let matrix = [1, 0, 0, 1, -window.innerWidth, 0];
            this.$emit('transform', {
                el,
                progress,
                matrix ( fn ) {
                    matrix = fn.call(null, matrix.slice()) || matrix;
                },
            });
            el.style.cssText = css({
                'transform': css.fn('matrix', matrix),
                'transition': `transform ${ duration }ms ${ bezier }`,
                'pointer-events': pointerEvents,
            });
            const once = () => {
                this.$once('tantan:transitionend', ( $el ) => {
                    if (el === $el) {
                        this.$emit('slideleftend');
                    } else {
                        once();
                    }
                });
            };
            once();
        });
        this.$on('tantan:slideright', () => {
            const { progress } = this;
            const duration = this.escapeSpeed;
            const bezier = this.escapeTimingFunction;
            const pointerEvents = 'none';
            const { items } = this.$refs;
            const el = items[items.length - 1];
            this.$emit('slideright');
            let matrix = [1, 0, 0, 1, window.innerWidth, 0];
            this.$emit('transform', {
                el,
                progress,
                matrix ( fn ) {
                    matrix = fn.call(null, matrix.slice()) || matrix;
                },
            });
            el.style.cssText = css({
                'transform': css.fn('matrix', matrix),
                'transition': `transform ${ duration }ms ${ bezier }`,
                'pointer-events': pointerEvents,
            });
            const once = () => {
                this.$once('tantan:transitionend', ( $el ) => {
                    if (el === $el) {
                        this.$emit('sliderightend');
                    } else {
                        once();
                    }
                });
            };
            once();
        });
        this.$on('tantan:slideback', () => {
            const { progress } = this;
            const duration = this.bounceSpeed;
            const bezier = this.bounceTimingFunction;
            const pointerEvents = 'auto';
            const { items } = this.$refs;
            const el = items[items.length - 1];
            this.$emit('slideback');
            let matrix = [1, 0, 0, 1, 0, 0];
            this.$emit('transform', {
                el,
                progress,
                matrix ( fn ) {
                    matrix = fn.call(null, matrix.slice()) || matrix;
                },
            });
            el.style.cssText = css({
                'transform': css.fn('matrix', matrix),
                'transition': `transform ${ duration }ms ${ bezier }`,
                'pointer-events': pointerEvents,
            });
            const once = () => {
                this.$once('tantan:transitionend', ( $el ) => {
                    if (el === $el) {
                        this.$emit('slidebackend');
                    } else {
                        once();
                    }
                });
            };
            once();
        });
    },
    methods: {
        // =prevpoint
        // =startpoint
        // =originpoint
        // setProgress()
        // @transform => pass the progress value
        start ( event, el ) {
            const { targetTouches } = event;
            const [ touch ] = slice(targetTouches);
            this.originpoint = {
                x: 0,
                y: 0,
            };
            this.startpoint = touch;
            this.prevpoint = touch;
            const { transform } = getComputedStyle(el);
            el.style.cssText = css({
                'transform': transform,
            });
            if (/matrix/i.test(transform)) {
                const [ a, b, c, d, e, f ] = transform.slice(7, -1).replace(/\s/g, '').split(',');
                this.originpoint.x = -e;
                this.originpoint.y = -f;
            }
            this.setProgress();
            const { progress } = this;
            this.$emit('transform', {
                el,
                progress,
                matrix () {},
            });
        },
        // =prevpoint
        // setProgress()
        // @transform => design draging animation
        move ( event, el ) {
            const { targetTouches } = event;
            const [ touch ] = slice(targetTouches);
            const { x: ox, y: oy } = this.originpoint;
            const { clientX: sx, clientY: sy } = this.startpoint;
            const { clientX: px, clientY: py } = this.prevpoint;
            const { clientX: nx, clientY: ny } = touch;
            let defaults = [1, 0, 0, 1, nx - sx - ox, ny - sy - oy];
            this.prevpoint = touch;
            this.setProgress();
            const { progress } = this;
            this.$emit('transform', {
                el,
                progress,
                matrix ( fn ) {
                    defaults = fn.call(null, defaults.slice()) || defaults;
                },
            });
            el.style.cssText = css({
                'transform': css.fn('matrix', defaults),
            });
        },
        // =progress
        setProgress () {
            const { x: ox, y: oy } = this.originpoint;
            const { clientX: sx, clientY: sy } = this.startpoint;
            const { clientX: px, clientY: py } = this.prevpoint;
            this.progress = (px - sx - ox) / this.escapeThreshold;
        },
        // @drag
        drag ( event, el ) {
            const { progress } = this;
            this.$emit('drag', { event, el, progress });
        },
        // @drop
        // @tantan:slideback
        // @tantan:slideleft
        // @tantan:slideright
        drop ( event, el ) {
            this.startpoint = null;
            this.prevpoint = null;
            const { progress } = this;
            this.$emit('drop', { event, el, progress });
            if (Math.abs(progress) < 1) {
                this.$emit('tantan:slideback');
            } else {
                if (progress < 0) {
                    this.$emit('tantan:slideleft');
                } else {
                    this.$emit('tantan:slideright');
                }
            }
        },
        // start()
        // @touchstart
        // drag()
        touchstart ( event, index ) {
            if (this.disabled) {
                return;
            }
            const el = this.$refs.items[index];
            this.start(event, el);
            this.$emit('touchstart', { event, el });
            this.drag(event, el);
        },
        // move()
        // @touchmove
        // drag()
        touchmove ( event, index ) {
            if (this.disabled) {
                return;
            }
            const el = this.$refs.items[index];
            this.move(event, el);
            this.$emit('touchmove', { event, el });
            this.drag(event, el);
        },
        // end()
        // @touchend
        // drop()
        touchend ( event, index ) {
            if (this.disabled) {
                return;
            }
            const el = this.$refs.items[index];
            this.$emit('touchend', { event, el });
            this.drop(event, el);
        },
        // end()
        // @touchcancel
        // drop()
        touchcancel ( event, index ) {
            if (this.disabled) {
                return;
            }
            const el = this.$refs.items[index];
            this.$emit('touchcancel', { event, el });
            this.drop(event, el);
        },
        // @tantan:transitionend
        transitionend ( event, index ) {
            const el = this.$refs.items[index];
            this.$emit('tantan:transitionend', el);
        },
    },
};
</script>

<template>
    <div class="tantan">
        <div class="tantan-item"
            ref="items"
            v-for="(item, index) in data"
            @touchstart="touchstart($event, index)"
            @touchmove.prevent="touchmove($event, index)"
            @touchend="touchend($event, index)"
            @touchcancel="touchcancel($event, index)"
            @transitionend.self="transitionend($event, index)"
        >
            <!-- export item and index -->
            <slot name="card"
                :item="item"
                :index="length - index - 1"
            ></slot>
        </div>
    </div>
</template>

<style lang="scss" scoped>
    .tantan {
        height: 100%;
        position: relative;
    }
    .tantan-item {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }
</style>
