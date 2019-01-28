<template>
    <div class="image-comparer-container"
         :class="`image-comparer-${orientation}`"
         :style="containerStyle"
         @touchstart="containerClick"
         @mousedown="containerClick">
        <img :src="after" alt="after"
             @mousedown.prevent
             :style="imgStyle.after"
             @load="setDimensions"/>
        <img :src="before" alt="before"
             @mousedown.prevent
             :style="imgStyle.before"/>
        <div class="image-comparer-overlay">
            <div v-if="beforeLabel" class="image-comparer-before-label">{{ beforeLabel }}</div>
            <div v-if="afterLabel" class="image-comparer-after-label">{{ afterLabel }}</div>
        </div>
        <div class="image-comparer-handle bg-primary"
             :class="`image-comparer-handle-${orientation}`"
             :style="handleStyle"
             @touchstart="startSlide"
             @mousedown="startSlide">
            <div :class="`image-comparer-arrow image-comparer-arrow-${beforeDirection}`"></div>
            <div :class="`image-comparer-arrow image-comparer-arrow-${afterDirection}`"></div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'image-comparer',
    props: {
        before: {
            type: String,
            required: true
        },
        beforeLabel: {
            type: String
        },
        after: {
            type: String,
            required: true
        },
        afterLabel: {
            type: String
        },
        offset: {
            type: [String, Number],
            default: 0.5,
            validator: (value) => {
                return (value >= 0 && value <= 1)
            }
        },
        clickToMove: {
            type: Boolean,
            default: true
        },
        handleWidth: {
            type: Number
        },
        handleHeight: {
            type: Number
        },
        orientation: {
            type: String,
            default: 'horizontal',
            validator: value => {
                return (value === 'horizontal' || value === 'vertical')
            }
        }
    },
    data() {
        return {
            imgOffset: null,
            slideOffset: { x: this.offset, y: this.offset },
            sliding: false,
            containerStyle: {},
            overlayStyle: {},
            handle: {
                width: this.handleWidth ? this.handleWidth : (this.orientation === 'horizontal' ? 60 : 30),
                height: this.handleHeight ? this.handleHeight : (this.orientation === 'horizontal' ? 30 : 60)
            },
            beforeDirection: (this.orientation === 'vertical') ? 'down' : 'left',
            afterDirection: (this.orientation === 'vertical') ? 'up' : 'right'
        }
    },
    methods: {
        setDimensions() {
            const img = this.$el.querySelector('img')
            this.imgOffset = img.getBoundingClientRect()
            this.containerStyle = { width: `${this.calcOffset.w}px`, height: `${this.calcOffset.h}px` }
        },
        containerClick(event) {
            if (this.clickToMove) {
                this.startSlide(event)
            }
        },
        startSlide(event) {
            this.sliding = true
            this.moveSlide(event)
        },
        moveSlide(event) {
            if (this.sliding) {
                let x = (event.touches ? event.touches[0].pageX : event.pageX) - this.imgOffset.left
                let y = (event.touches ? event.touches[0].pageY : event.pageY) - this.imgOffset.top
                x = (x < 0) ? 0 : ((x > this.calcOffset.w) ? this.calcOffset.w : x) / this.calcOffset.w
                y = (y < 0) ? 0 : ((y > this.calcOffset.h) ? this.calcOffset.h : y) / this.calcOffset.h
                this.slideOffset = { x, y }
            }
        },
        endSlide() {
            this.sliding = false
        },
        resize() {
            this.containerStyle = {}
            this.$nextTick(() => this.setDimensions())
        }
    },
    computed: {
        imgStyle() {
            let after, before
            if (this.orientation === 'horizontal') {
                after = { clip: `rect(0, ${this.calcOffset.w}px, ${this.calcOffset.h}px, ${this.calcOffset.cw}px)` }
                before = { clip: `rect(0, ${this.calcOffset.cw}px, ${this.calcOffset.h}px, 0)` }
            } else {
                after = { clip: `rect(${this.calcOffset.ch}px, ${this.calcOffset.w}px, ${this.calcOffset.h}px, 0)` }
                before = { clip: `rect(0, ${this.calcOffset.w}px, ${this.calcOffset.ch}px, 0)` }
            }
            return { after, before }
        },
        handleStyle() {
            let width = this.handle.width
            let height = this.handle.height
            const horizontal = this.orientation === 'horizontal'
            return {
                width: `${width}px`,
                height: `${height}px`,
                top: horizontal ? `calc(50% - ${height / 2}px)` : `calc(${this.slideOffset.y * 100}% - ${height / 2}px)`,
                left: horizontal ? `calc(${this.slideOffset.x * 100}% - ${width / 2}px)` : `calc(50% - ${width / 2}px)`
            }
        },
        calcOffset() {
            let w = 0
            let h = 0
            if (this.imgOffset) {
                w = this.imgOffset.width
                h = this.imgOffset.height
            }
            let cw = this.slideOffset.x * w
            let ch = this.slideOffset.y * h
            return { w, h, cw, ch }
        }
    },
    mounted() {
        document.addEventListener('touchmove', this.moveSlide)
        document.addEventListener('touchend', this.endSlide)
        document.addEventListener('mousemove', this.moveSlide)
        document.addEventListener('mouseup', this.endSlide)
        window.addEventListener('resize', this.resize)
    },
    beforeDestroy() {
        document.removeEventListener('touchmove', this.moveSlide)
        document.removeEventListener('touchend', this.endSlide)
        document.removeEventListener('mousemove', this.moveSlide)
        document.removeEventListener('mouseup', this.endSlide)
        window.removeEventListener('resize', this.resize)
    }
}
</script>

<style lang="scss" scoped>
    .image-comparer-container {
        position: relative;
        overflow: hidden;
        box-sizing: content-box;

        img {
            max-width: 100%;
            position: absolute;
            top: 0;
            left: 0;
            display: block;
            user-select: none;
            z-index: 20;
        }

        .image-comparer-overlay {
            z-index: 25;
            width: 100%;
            height: 100%;
            top: 0;
            position: absolute;

            .image-comparer-before-label, .image-comparer-after-label {
                user-select: none;
                position: absolute;
                font-size: 0.8em;
                top: calc(50% - 0.4em - 5px);
                padding: 10px;
                background: rgba(255, 255, 255, 0.4);
                color: white;
            }

            .image-comparer-before-label {
                left: 0;
            }

            .image-comparer-after-label {
                right: 0;
            }
        }

        &:hover > .image-comparer-overlay {
            opacity: 1;
        }

        .image-comparer-handle {
            cursor: move;
            z-index: 30;
            position: absolute;
            border-radius: 30%;
            user-select: none;
            display: flex;
            align-items: center;
            justify-content: space-between;

            &.image-comparer-handle-horizontal {

            }
            &.image-comparer-handle-vertical {
                flex-direction: column;
            }
        }

        .image-comparer-handle:before, .image-comparer-handle:after {
            content: "";
            border: 2px solid white;
            position: absolute;
            z-index: 29;
        }

        &.image-comparer-horizontal {
            .image-comparer-handle:before, .image-comparer-handle:after {
                height: 9999px;
                left: calc(50%);
            }

            .image-comparer-handle:before {
                top: 100%;
            }

            .image-comparer-handle:after {
                bottom: 100%;
            }

            .image-comparer-arrow-right {
                border-color: transparent transparent transparent white;
            }

            .image-comparer-arrow-left {
                border-color: transparent white transparent transparent;
            }
        }

        &.image-comparer-vertical {
            .image-comparer-handle:before, .image-comparer-handle:after {
                width: 9999px;
                top: calc(50%);
            }

            .image-comparer-handle:before {
                left: 100%;
            }

            .image-comparer-handle:after {
                right: 100%;
            }

            .image-comparer-arrow-down {
                border-color: transparent transparent white transparent;
            }

            .image-comparer-arrow-up {
                border-color: white transparent transparent transparent;
            }
        }

        .image-comparer-arrow {
            user-select: none;
            position: relative;
            width: 0;
            height: 0;
            border: 8px inset transparent;
        }
    }
</style>
