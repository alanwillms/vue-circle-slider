<template>
  <div>
    <svg :width="side + 'px'" :height="side + 'px'" :viewBox="'0 0 ' + side + ' ' + side" ref="_svg"
      @touchmove="handleTouchMove"
      @click="handleClick"
      @mousedown="handleMouseDown"
      @mouseup="handleMouseUp"
    >
      <g>
        <path :stroke="circleColor" fill="none" :stroke-width="cpMainCircleStrokeWidth" :d="cpPathD(cpStartAngle, cpEndAngle, 1)"></path>
        <path :stroke="progressColor" fill="none" :stroke-width="cpPathStrokeWidth" :d="cpPathD(cpOriginAngle, cpAngle, cpPathDirection)"></path>
        <circle :fill="knobColor" :r="cpKnobRadius" :cx="cpPathX" :cy="cpPathY"></circle>
      </g>
    </svg>
  </div>
</template>
<script>
import TouchPosition from '../modules/touch_position.js'
import CircleSliderState from '../modules/circle_slider_state.js'
export default {
  name: 'CircleSlider',
  created () {
    this.stepsCount = 1 + (this.max - this.min) / this.stepSize
    this.steps = Array.from({
      length: this.stepsCount
    }, (_, i) => this.min + i * this.stepSize)

    this.circleSliderState = new CircleSliderState(this.steps, 0, this.value, this.arcLengthRadians)
    this.angle = this.circleSliderState.angleValue
    this.currentStepValue = this.circleSliderState.currentStep

    this.originValue = this.origin === null ? this.min : this.origin
    this.originValue = Math.min(this.max, Math.max(this.min, this.originValue))

    this.updateFromPropValue(this.value)
  },

  mounted () {
    this.createTouchPosition()
  },

  updated () {
    this.createTouchPosition()
  },

  props: {
    value: {
      type: Number,
      required: false,
      default: 0
    },
    side: {
      type: Number,
      required: false,
      default: 100
    },
    stepSize: {
      type: Number,
      required: false,
      default: 1
    },
    min: {
      type: Number,
      required: false,
      default: 0
    },
    max: {
      type: Number,
      required: false,
      default: 100
    },
    circleColor: {
      type: String,
      required: false,
      default: '#334860'
    },
    progressColor: {
      type: String,
      required: false,
      default: '#00be7e'
    },
    knobColor: {
      type: String,
      required: false,
      default: '#00be7e'
    },
    knobRadius: {
      type: Number,
      required: false,
      default: null
    },
    knobRadiusRel: {
      type: Number,
      required: false,
      default: 7
    },
    circleWidth: {
      type: Number,
      required: false,
      default: null
    },
    circleWidthRel: {
      type: Number,
      required: false,
      default: 20
    },
    progressWidth: {
      type: Number,
      required: false,
      default: null
    },
    progressWidthRel: {
      type: Number,
      required: false,
      default: 10
    },
    arcLengthDegrees: {
      type: Number,
      required: false,
      default: 360
    },
    arcOffsetDegrees: {
      type: Number,
      required: false,
      default: 0
    },
    origin: {
      type: Number,
      required: false,
      default: null
    }
    // limitMin: {
    //   type: Number,
    //   required: false,
    //   default: null
    // },
    // limitMax: {
    //   type: Number,
    //   required: false,
    //   default: null
    // }
  },
  data () {
    return {
      originValue: null,
      steps: null,
      stepsCount: null,
      angle: 0,
      currentStepValue: 0,
      mousePressed: false,
      circleSliderState: null,
      mousemoveTicks: 0
    }
  },
  computed: {
    // cpStartAngleOffset () {
    //   if (!this.minStepLimit) {
    //     return 0
    //   }
    // },
    cpCenter () {
      return this.side / 2
    },
    cpAngle () {
      return this.angle + this.arcOffsetRadians
    },
    cpMainCircleStrokeWidth () {
      return this.circleWidth || (this.side / 2) / this.circleWidthRel
    },
    cpPathDirection () {
      return (this.cpAngle <= this.cpOriginRadians + this.arcOffsetRadians) ? 0 : 1
    },
    cpPathX () {
      return this.pathX(this.cpAngle)
    },
    cpPathY () {
      return this.pathY(this.cpAngle)
    },
    cpStartAngle () {
      return this.arcOffsetRadians
    },
    cpEndAngle () {
      return (this.arcLengthRadians + this.arcOffsetRadians)
    },
    cpOriginRadians () {
      return this.circleSliderState.angleUnit * (this.originValue - this.min)
    },
    cpOriginAngle () {
      return this.arcOffsetRadians + this.cpOriginRadians
    },
    cpPathStrokeWidth () {
      return this.progressWidth || (this.side / 2) / this.progressWidthRel
    },
    cpKnobRadius () {
      return this.knobRadius || (this.side / 2) / this.knobRadiusRel
    },
    radius () {
      let maxCurveWidth = Math.max(this.cpMainCircleStrokeWidth, this.cpPathStrokeWidth)
      return (this.side / 2) - Math.max(maxCurveWidth, this.cpKnobRadius * 2) / 2
    },
    arcLengthRadians () {
      return this.arcLengthDegrees * Math.PI * 2 / 360
    },
    arcOffsetRadians () {
      return this.arcOffsetDegrees * Math.PI * 2 / 360
    }
  },
  methods: {

    createTouchPosition () {
      this.touchPosition = new TouchPosition(this.$refs._svg, this.radius, this.radius / 2)
    },

    cpPathD (startAngle, endAngle, direction) {
      let parts = []

      let startX = this.pathX(startAngle)
      let startY = this.pathY(startAngle)

      parts.push('M' + startX)
      parts.push(startY)

      while (Math.abs(endAngle - startAngle) >= Math.PI) {
        let angle = startAngle + Math.PI
        var endX = this.pathX(angle)
        var endY = this.pathY(angle)

        parts.push('A')
        parts.push(this.radius)
        parts.push(this.radius)
        parts.push(0)
        parts.push(1)
        parts.push(direction)
        parts.push(endX)
        parts.push(endY)

        startAngle += (endAngle > startAngle ? 1 : -1) * Math.PI
      }

      endX = this.pathX(endAngle)
      endY = this.pathY(endAngle)
      parts.push('A')
      parts.push(this.radius)
      parts.push(this.radius)
      parts.push(0)
      parts.push(Math.abs(endAngle - startAngle) >= Math.PI ? 1 : 0)
      parts.push(direction)
      parts.push(endX)
      parts.push(endY)

      return parts.join(' ')
    },

    pathX (angle) {
      return this.cpCenter + this.radius * Math.cos(angle)
    },

    pathY (angle) {
      return this.cpCenter + this.radius * Math.sin(angle)
    },

    /*
     */
    fitToStep (val) {
      return Math.round(val / this.stepSize) * this.stepSize
    },

    /*
     */
    handleClick (e) {
      this.touchPosition.setNewPosition(e)
      if (this.touchPosition.isTouchWithinSliderRange) {
        const newAngle = this.calculateAngle()
        this.animateSlider(this.angle, newAngle)
      }
    },

    /*
     */
    handleMouseDown (e) {
      e.preventDefault()
      this.mousePressed = true
      window.addEventListener('mousemove', this.handleWindowMouseMove)
      window.addEventListener('mouseup', this.handleMouseUp)
    },

    /*
     */
    handleMouseUp (e) {
      e.preventDefault()
      this.mousePressed = false
      window.removeEventListener('mousemove', this.handleWindowMouseMove)
      window.removeEventListener('mouseup', this.handleMouseUp)
      this.mousemoveTicks = 0
    },

    /*
     */
    handleWindowMouseMove (e) {
      e.preventDefault()
      if (this.mousemoveTicks < 5) {
        this.mousemoveTicks++
        return
      }

      this.touchPosition.setNewPosition(e)
      this.updateSlider()
    },

    /*
     */
    handleTouchMove (e) {
      this.$emit('touchmove')
      // Do nothing if two or more fingers used
      if (e.targetTouches.length > 1) {
        return true
      }

      const lastTouch = e.targetTouches.item(e.targetTouches.length - 1)
      this.touchPosition.setNewPosition(lastTouch)

      if (this.touchPosition.isTouchWithinSliderRange) {
        e.preventDefault()
        this.updateSlider()
      }
    },

    calculateAngle () {
      var angle = (this.touchPosition.sliderAngle - this.arcOffsetRadians + Math.PI * 2) % (Math.PI * 2)
      var angleMod = this.angle % (Math.PI * 2)
      var loops = Math.trunc(this.angle / (Math.PI * 2))
      if (angle < Math.PI / 2 && angleMod > 1.5 * Math.PI && this.angle < this.arcLengthDegrees) {
        loops += 1
      } else if (angleMod < Math.PI / 2 && angle > 1.5 * Math.PI) {
        loops -= 1
      }

      loops = Math.max(0, loops)
      angle = angle + loops * Math.PI * 2

      return angle
    },

    /*
     */
    updateAngle (angle) {
      this.circleSliderState.updateCurrentStepFromAngle(angle)
      this.angle = this.circleSliderState.angleValue
      this.currentStepValue = this.circleSliderState.currentStep

      this.$emit('input', this.currentStepValue)
    },

    /*
     */
    updateFromPropValue (value) {
      let stepValue = this.fitToStep(value)
      this.circleSliderState.updateCurrentStepFromValue(stepValue)

      this.angle = this.circleSliderState.angleValue
      this.currentStepValue = stepValue
      this.$emit('input', this.currentStepValue)
    },

    /*
     */
    updateSlider () {
      const angle = this.calculateAngle()
      if (Math.abs(this.angle - angle) < Math.PI) {
        this.updateAngle(Math.max(0.0, Math.min(angle, this.arcLengthRadians)))
      }
    },

    /*
     */
    animateSlider (startAngle, endAngle) {
      const direction = startAngle < endAngle ? 1 : -1
      const curveAngleMovementUnit = direction * this.circleSliderState.angleUnit * 2

      const animate = () => {
        if (Math.abs(endAngle - startAngle) < Math.abs(2 * curveAngleMovementUnit)) {
          this.updateAngle(endAngle)
        } else {
          const newAngle = startAngle + curveAngleMovementUnit
          this.updateAngle(newAngle)
          this.animateSlider(newAngle, endAngle)
        }
      }

      window.requestAnimationFrame(animate)
    }
  },
  watch: {
    value (val) {
      this.updateFromPropValue(val)
    }
  }
}
</script>
