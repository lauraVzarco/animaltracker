<script>
import UserDefinedField from "./UserDefinedField";
export default {
  components: { UserDefinedField },
  props: {
    metadata: {
      type: Object,
      required: true
    },
    listOfAllMetadata
  },
  methods: {
    toggleExifField(field) {
      if (this.metadata.selectedExifData.includes(field)) {
        const index = this.metadata.selectedExifData.indexOf(field);
        this.metadata.selectedExifData.splice(index, 1);
      } else {
        this.metadata.selectedExifData.push(field);
      }
    },
    toggleUserDataField(field) {
      if (this.metadata.selectedUserData.includes(field)) {
        const index = this.metadata.selectedUserData.indexOf(field);
        this.metadata.selectedUserData.splice(index, 1);
      } else {
        this.metadata.selectedUserData.push(field);
      }
    },
    addField(key) {
      this.$set(this.metadata.userData, key, "");
      this.metadata.selectedUserData.push(key);
    },
    hideUnselectedFields() {},
    changeUserSelectedValue(key, value) {
      this.$set(this.metadata.userData, key, value);
    }
  },
  render(h) {
    return (
      <div class="metadataContainer">
        <div class="title">Image Metadata for {this.metadata.name}</div>
        <div>
          <button onClick={this.hideUnselectedFields}>
            Hide all Unselected Fields
          </button>
          {Object.entries(this.metadata.exifdata).map(([key, value]) => {
            return (
              <div>
                <div class="metadata-fields">
                  <input
                    type="checkbox"
                    checked={this.metadata.selectedExifData.includes(key)}
                    onInput={() => this.toggleExifField(key)}
                  />
                  <div class="key">{key}:</div> <div class="value">{value}</div>
                </div>
              </div>
            );
          })}
          {Object.entries(this.metadata.userData).map(([key, value]) => {
            return (
              <div>
                <div class="metadata-fields">
                  <input
                    type="checkbox"
                    checked={this.metadata.selectedUserData.includes(key)}
                    onInput={() => this.toggleUserDataField(key)}
                  />
                  <div class="key">{key}:</div>{" "}
                  <input
                    type="text"
                    onInput={e =>
                      this.changeUserSelectedValue(key, e.target.value)
                    }
                    value={value}
                  />
                </div>
              </div>
            );
          })}
        </div>
        <UserDefinedField addField={this.addField} />
      </div>
    );
  }
};
</script>
<style>
.metadataContainer {
  width: 300px;
  height: 405px;
  overflow: scroll;
  background: #aaaa;
  padding: 10px;
  margin-right: 50px;
  align-content: center;
}
.title {
  text-align: center;
  font-size: 14px;
  margin-bottom: 10px;
  font-weight: bold;
}

.metadata-fields {
  display: flex;
  flex-direction: row;
}

.key {
  font-weight: bold;
  margin-left: 6px;
  margin-right: 6px;
}
.value {
  margin-right: 6px;
}
</style>