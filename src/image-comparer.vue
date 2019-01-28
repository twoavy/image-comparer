<template>
    <div class="image-comparer-container"
         :style="containerStyle"
         @touchstart="startSlide"
         @mousedown="startSlide">
        <img :src="after" alt="after"
             @mousedown.prevent
             @load="setDimensions"/>
        <img :src="before" alt="before"
             @mousedown.prevent
             :style="beforeImgStyle"/>
        <div class="image-comparer-overlay">
            <div v-if="beforeLabel" class="image-comparer-before-label">{{ beforeLabel }}</div>
            <div v-if="afterLabel" class="image-comparer-after-label">{{ afterLabel }}</div>
        </div>
        <div class="image-comparer-handle bg-primary"
             :style="handleStyle">
            <div class="image-comparer-arrow-left"></div>
            <div class="image-comparer-arrow-right"></div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'image-comparer',
    data() {
        return {
            imgOffset: null,
            slideOffset: this.offset,
            sliding: false,
            containerStyle: {},
            overlayStyle: {}
        }
    },
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
                return (value > 0 && value <= 1)
            }
        }
    },
    methods: {
        setDimensions() {
            const img = this.$el.querySelector('img')
            this.imgOffset = img.getBoundingClientRect()
            this.containerStyle = { width: `${this.w}px`, height: `${this.h}px` }
        },
        startSlide(event) {
            this.sliding = true
            this.moveSlide(event)
        },
        moveSlide(event) {
            if (this.sliding) {
                let x = (event.touches ? event.touches[0].pageX : event.pageX) - this.imgOffset.left
                x = (x < 0) ? 0 : ((x > this.w) ? this.w : x)
                this.slideOffset = (x / this.w)
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
        beforeImgStyle() {
            return { clip: `rect(0, ${this.x}px, ${this.h}px, 0)` }
        },
        handleStyle() {
            const width = 60
            const height = 30
            return {
                width: `${width}px`,
                height: `${height}px`,
                top: `calc(50% - ${height / 2}px)`,
                left: `calc(${this.slideOffset * 100}% - ${width / 2}px)`
            }
        },
        x() {
            return this.w * this.slideOffset
        },
        w() {
            if (this.imgOffset) {
                return this.imgOffset.width
            }
            return null
        },
        h() {
            if (this.imgOffset) {
                return this.imgOffset.height
            }
            return null
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
            padding: 0 5px;
            align-items: center;
            justify-content: space-between;
        }

        .image-comparer-handle:before, .image-comparer-handle:after {
            content: "";
            border: 2px solid white;
            height: 9999px;
            position: absolute;
            left: calc(50% - 5px);
            z-index: 29;
        }

        .image-comparer-handle:before {
            top: 100%;
        }

        .image-comparer-handle:after {
            bottom: 100%;
        }

        .image-comparer-arrow-right,
        .image-comparer-arrow-left {
            user-select: none;
            position: relative;
            width: 0;
            height: 0;
            border: 8px solid;
        }

        .image-comparer-arrow-right {
            border-color: transparent transparent transparent white;
        }

        .image-comparer-arrow-left {
            border-color: transparent white transparent transparent;
        }
    }
</style>
