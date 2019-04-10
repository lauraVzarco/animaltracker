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
        <div class="user-metadata">
          <input
            class="input"
            type="text"
            value={this.key}
            onInput={e => {
              this.key = e.target.value;
            }}
          />
          :
          <input
            class="input"
            type="text"
            value={this.value}
            onInput={e => {
              this.value = e.target.value;
            }}
          />
        </div>
        <button
          class="add-metadata"
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

<style>
.user-metadata {
  margin-top: 15px;
  margin-bottom: 10px;
  display: flex;
  justify-content: center;
}

.input {
  width: 30%;
  height: 20px;
}

.add-metadata {
  margin-bottom: 15px;
  display: flex;
  justify-content: center;
  margin: 0 auto;
}
</style>

