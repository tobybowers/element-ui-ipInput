<template>
  <div class="el-input ip-input" :class="inputSize">
    <input
      class="el-input__inner"
      :key="index"
      min="0"
      max="255"
      pattern="^[0-9].{0,3}"
      ref="ipSegment"
      step="1"
      type="number"
      v-for="(segment, index) in ipArray"
      v-model="ipArray[index]"
      @blur="blur"
      @focus="ipFocus(index)"
      @keydown="ipKeydown($event, index)"
      @paste="paste($event)"
    />
  </div>
</template>

<script>

export default {
  name: 'ipInput',
  props: {
    ipAddress: {
      required: true,
      type: String
    },
    size: {
      default: '',
      required: false,
      type: String,
      validator: function (value) {
        // The value must match one of these strings
        return ['', 'medium', 'small', 'mini'].indexOf(value) !== -1
      }
    }
  },
  data () {
    return {
      ipArray: ['', '', '', ''],
      valid: false,
      active: false,
      timeout: null
    }
  },
  beforeMount () {
    this.propToData(this.ipAddress)
  },
  computed: {
    inputSize () {
      switch (this.size) {
        case 'medium':
          return 'el-input--medium'
        case 'small':
          return 'el-input--small'
        case 'mini':
          return 'el-input--mini'
        default:
          return ''
      }
    }
  },
  methods: {
    /**
    * Convert the array of IP segments back to a string
    */
    arrayToIp (ipArray) {
      return ipArray.join('.')
    },
    /**
    * Clicked off the IP or port
    */
    blur () {
      this.active = false
    },
    /**
    * Update the controller with changed IP and port addresses
    */
    changed (ip = this.ipArray) {
      if (this.validateIP(ip)) {
        this.valid = true
        this.$emit('update', this.arrayToIp(ip))
      } else {
        this.valid = false
      }
    },
    /**
    * Clear both inputs
    */
    clearAll () {
      this.ipArray = ['', '', '', '']
      this.valid = false
    },
    /**
     * On focus clear the current box
     */
    ipFocus (index) {
      this.active = true
    },
    /**
     * Run on keydown
     */
    ipKeydown (event, index) {
      //  clearTimeout(this.timeout);
      let keyCode = event.keyCode || event.which
      // Backspace or left on keypad
      if (keyCode === 8 || keyCode === 37) {
        // If there is nothing within the selected input go the the one before it
        if (this.ipArray[index].length === 0 && this.ipArray[index - 1] !== undefined) this.$refs.ipSegment[index - 1].focus()
      }
      // Allow tabing to next segment using Period or Decimal point if current segment has entry
      else if (keyCode === 110 || keyCode === 190) {
        event.preventDefault()
        if (this.ipArray[index].length > 0) this.moveToNextIpSegment(index, true)
      }
      // Semi-colon (jump to port number)
      else if (keyCode === 186) {
        this.$refs.portSegment.focus()
      }
      setTimeout(() => {
        // If its a 0 then always move to the next segment
        if (this.ipArray[index] === '0' || this.ipArray[index].length === 3) this.moveToNextIpSegment(index, true)
        else this.moveToNextIpSegment(index, false)
        // Update the change
        this.changed()
      })
    },
    /**
    * Convert the IP address string to an array of values
    */
    ipToArray (ip) {
      let segments = []
      ip.split('.').map(segment => {
        if (isNaN(segment) || segment < 0 || segment > 255) {
          segment = 255
        }
        segments.push(segment)
      })
      // If something is not valid clear it all
      if (segments.length !== 4) {
        //console.error('Not valid, so clearing ip', segments);
        this.clearAll()
      } else {
        this.ipArray = segments
      }
    },
    /**
    * Paste in an IP (with or without a port)
    */
    paste (event) {
      // Focus on first el
      this.$refs.ipSegment[0].focus()
      // Get clipboard text
      let pasteText = event.clipboardData.getData('text/plain')
      this.propToData(pasteText)
      this.changed()
      // Blur off input
      this.$refs.ipSegment[0].blur()
    },
    /**
     * Copy prop into local copy
     */
    propToData (ip) {
      if (ip) this.ipToArray(ip)
    },
    /**
    * Work out if we need to move to the next IP segment or not
    */
    moveToNextIpSegment (index, next = true) {
      // If there is 3 characters or 0 value check if there is another segment, if there is focus on it.
      if (next) {
        if (this.ipArray[index + 1] !== undefined) {
          this.$refs.ipSegment[index + 1].focus()
        }
        else if (this.ipArray[index + 1] === undefined) {
          if((index + 1) < 4) {
            this.$refs.ipSegment[index + 1].focus()
          }
        }
      } else {
        if (this.ipArray[index].length >= 3 && this.ipArray[index + 1] !== undefined) {
          this.$refs.ipSegment[index + 1].focus()
        }
      }
    },
    /**
    * validates the IP address
    *
    * @returns Boolean
    */
    validateIP (ip) {
      let ipCheck = this.arrayToIp(ip)
      return (/^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/.test(ipCheck))
    }
  },
  watch: {
    ipAddress (newIp) {
      this.propToData(newIp)
    }
  }
}
</script>

<style lang="scss" scoped>
.ip-input {
  input {
    width: 55px;
    &:first-child {
      border-top-right-radius: unset;
      border-bottom-right-radius: unset;
    }
    &:last-child {
      border-top-left-radius: unset;
      border-bottom-left-radius: unset;
    }
    &:not(:first-child):not(:last-child) {
      border-radius: unset;
    }
  }
  input[type="number"]::-webkit-outer-spin-button, input[type="number"]::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
  input[type="number"] {
    -moz-appearance: textfield;
  }
}
</style>
