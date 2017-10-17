<script>
function css ( obj ) {
    return Object.keys(obj).map(( key ) => {
        const value = obj[key];
        return `${ key }:${ (value instanceof Array ? value.join('') : value) }`;
    }).join(';');
}

css.fn = function ( name, args ) {
    return `${ name }(${ args.join(',') })`;
};

export default {
    props: {
        bounceSpeed: {
            type: Number,
            default: 250,
        },
        bounceTimingFunction: {
            type: String,
            default: css.fn('cubic-bezier', [0, 0, 0.59, 1.12]),
        },
        escapeSpeed: {
            type: Number,
            default: 300,
        },
        escapeTimingFunction: {
            type: String,
            default: css.fn('cubic-bezier', [0, 0, 0.59, 1.12]),
        },
        escapeThreshold: {
            type: Number,
            default: 60,
        },
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
    },
    methods: {
        start ( event, el ) {
            const { targetTouches } = event;
            const [ touch ] = targetTouches;
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
        },
        move ( event, el ) {
            const { targetTouches } = event;
            const [ touch ] = targetTouches;
            const { x: ox, y: oy } = this.originpoint;
            const { clientX: sx, clientY: sy } = this.startpoint;
            const { clientX: px, clientY: py } = this.prevpoint;
            const { clientX: nx, clientY: ny } = touch;
            el.style.cssText = css({
                'transform': css.fn('matrix', [1, 0, 0, 1, nx - sx - ox, ny - sy - oy]),
            });
            this.prevpoint = touch;
            this.setProgress();
        },
        end ( event, el ) {
            const { x: ox, y: oy } = this.originpoint;
            const { clientX: sx, clientY: sy } = this.startpoint;
            const { clientX: px, clientY: py } = this.prevpoint;
            let bezier;
            let duration;
            let translateX;
            let pointerEvents;
            if (Math.abs(this.progress) >= 1) {
                duration = this.escapeSpeed;
                bezier = this.bounceTimingFunction;
                if (this.progress > 0) {
                    translateX = window.innerWidth
                } else {
                    translateX = window.innerWidth * -1;
                }
                pointerEvents = 'none;'
            } else {
                translateX = 0;
                duration = this.bounceSpeed;
                bezier = this.escapeTimingFunction;
                pointerEvents = 'auto';
            }
            el.style.cssText = css({
                'transform': css.fn('matrix', [1, 0, 0, 1, translateX, 0]),
                'transition': `transform ${ duration }ms ${ bezier }`,
                'pointer-events': pointerEvents,
            });
            this.startpoint = null;
            this.prevpoint = null;
        },
        setProgress () {
            const { x: ox, y: oy } = this.originpoint;
            const { clientX: sx, clientY: sy } = this.startpoint;
            const { clientX: px, clientY: py } = this.prevpoint;
            this.progress = (px - sx - ox) / this.escapeThreshold;
        },
        drag ( event, el ) {
            const { progress } = this;
            this.$emit('drag', { event, el, progress });
        },
        drop ( event, el ) {
            let type;
            const { progress } = this;
            this.$emit('drop', { event, el, progress });
            if (Math.abs(progress) < 1) {
                type = 'slideback';
            } else {
                if (progress < 0) {
                    type = 'slideleft';
                } else {
                    type = 'slideright';
                }
            }
            this.$emit(type, { event, el });
            let once = ( type ) => {
                this.$once('tantan:transitionend', ( $el ) => {
                    if (el === $el) {
                        this.$emit(`${ type }end`, { event, el });
                    } else {
                        once(type);
                    }
                });
            };
            once(type);
        },
        touchstart ( event, index ) {
            const el = this.$refs.items[index];
            this.start(event, el);
            this.$emit('touchstart', { event, el });
            this.drag(event, el);
        },
        touchmove ( event, index ) {
            const el = this.$refs.items[index];
            this.move(event, el);
            this.$emit('touchmove', { event, el });
            this.drag(event, el);
        },
        touchend ( event, index ) {
            const el = this.$refs.items[index];
            this.end(event, el);
            this.$emit('touchend', { event, el });
            this.drop(event, el);
        },
        touchcancel ( event, index ) {
            const el = this.$refs.items[index];
            this.end(event, el);
            this.$emit('touchcancel', { event, el });
            this.drop(event, el);
        },
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
            @touchmove="touchmove($event, index)"
            @touchend="touchend($event, index)"
            @touchcancel="touchcancel($event, index)"
            @transitionend.self="transitionend($event, index)"
        >
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
