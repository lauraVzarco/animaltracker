<script>
export default {
  props: {
    addField: {
      type: Function,
      required: true
    }
  },
  data() {
    return {
      key: "",
      value: ""
    };
  },
  render(h) {
    return (
      <div>
        <input
          type="text"
          value={this.key}
          onInput={e => {
            this.key = e.target.value;
          }}
        />
        :
        <input
          type="text"
          value={this.value}
          onInput={e => {
            this.value = e.target.value;
          }}
        />
        <button
          onClick={() => {
            this.addField(this.key, this.value);
            this.key = "";
            this.value = "";
          }}
        >
          add
        </button>
      </div>
    );
  }
};
</script>
