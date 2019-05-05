<script>
import UserDefinedField from "./UserDefinedField";
export default {
  components: { UserDefinedField },
  data() {
    return {
      exifdata: [{key: 'piwi', selected: true}, {key: 'chocu', selected: false}],
      userData: []
    }
  },
  props: {
    metadata: {
      type: Object,
      required: true
    },
  },
  methods: {
    toggleExifField(field) {
      const element = this.exifdata.find(el => el.key === field);
      this.$set(element, 'selected', !element.selected);
    },
    toggleUserDataField(field) {
      const element = this.userData.find(el => el.key === field);
      this.$set(element, 'selected', !element.selected);
    },
    addUserField(key) {
      // this.$set(this.metadata.userData, key, "");
      this.userData.push({key, selected: true});
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
            Hide/show all Unselected Fields
          </button>
          {Object.entries(this.exifdata).map(({key, selected}) => {
            return (
              <div>
                <div class="metadata-fields">
                  <input
                    type="checkbox"
                    checked={selected}
                    onInput={() => this.toggleExifField(key)}
                  />
                  <div class="key">{key}:</div> <div class="value">{value}</div>
                </div>
              </div>
            );
          })}
          {Object.entries(this.userData).map(({key, selected}) => {
            return (
              <div>
                <div class="metadata-fields">
                  <input
                    type="checkbox"
                    checked={selected}
                    onInput={() => this.toggleUserDataField(key)}
                  />
                  <div class="key">{key}:</div>{" "}
                  <input
                    type="text"
                    onInput={e =>
                      this.changeUserSelectedValue(key, e.target.value)
                    }
                    value={this.currentImage.userData[key] || ''}
                  />
                </div>
              </div>
            );
          })}
        </div>
        <UserDefinedField addField={this.addUserField} />
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