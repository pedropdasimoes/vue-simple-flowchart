<template>
  <div>
    <div
    style="display:flex;
    flex-direction: column;
    align-items:center;
    justify-content:center;
    height: fit-content">
      <h3 style="margin:0px;">Escala</h3>
      <div style="
      width: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      margin: 10px 0px;">
        <!-- <button @click="diminui">-</button> -->
        <button @click="diminui" class="button esquerda" type="button">
          -
        </button>
        <span style="
                padding: 0.12rem 0.5rem;
                border-top: 1px solid #ccc;
                border-bottom: 1px solid #ccc;
        ">{{scale}}</span>
        <button @click="aumenta" class="button direita" type="button">+</button>
      </div>
    </div>
    <div class="flowchart-container" @mousemove="handleMove" @mouseup="handleUp" @mousedown="handleDown">
      <div
        v-bind:style="{ transform: 'scale('+ scale + ')', marginLeft: '0px', marginTop: marginTopStyle, height:'5000px', width:'5000px' }">
        <svg width="100%" :height="`4000px`">
          <flowchart-link v-bind.sync="link" v-for="(link, index) in lines" :key="`link${index}`"
            @deleteLink="linkDelete(link.id)"></flowchart-link>
        </svg>
        <flowchart-node v-bind.sync="node" v-for="(node, index) in scene.nodes" :key="`node${index}`"
          :options="nodeOptions" @linkingStart="linkingStart(node.id)" @linkingStop="linkingStop(node.id)"
          @nodeSelected="nodeSelected(node.id, $event)">
        </flowchart-node>
      </div>
    </div>
  </div>
</template>

<script>
import FlowchartLink from "./FlowchartLink.vue";
import FlowchartNode from "./FlowchartNode.vue";
import { getMousePosition } from "../assets/utilty/position";

export default {
  name: "VueFlowchart",
  props: {
    scene: {
      type: Object,
      default() {
        return {
          centerX: 1024,
          scale: 1,
          centerY: 140,
          nodes: [],
          links: [],
        };
      },
    },
    height: {
      type: Number,
      default: 400,
    },
  },
  data() {
    return {
      scale: 1.0,
      marginLeftStyle: '0%',
      marginTopStyle: '0%',
      action: {
        linking: false,
        dragging: false,
        scrolling: false,
        selected: 0,
      },
      mouse: {
        x: 0,
        y: 0,
        lastX: 0,
        lastY: 0,
      },
      draggingLink: null,
      rootDivOffset: {
        top: 0,
        left: 0,
      },
    };
  },
  components: {
    FlowchartLink,
    FlowchartNode,
  },
  computed: {
    nodeOptions() {
      return {
        centerY: this.scene.centerY,
        centerX: this.scene.centerX,
        scale: this.scene.scale,
        offsetTop: this.rootDivOffset.top,
        offsetLeft: this.rootDivOffset.left,
        selected: this.action.selected,
      };
    },
    lines() {
      const lines = this.scene.links.map((link) => {
        const fromNode = this.findNodeWithID(link.from);
        const toNode = this.findNodeWithID(link.to);
        let x, y, cy, cx, ex, ey;
        x = this.scene.centerX + fromNode.x;
        y = this.scene.centerY + fromNode.y;
        [cx, cy] = this.getPortPosition("bottom", x, y);
        x = this.scene.centerX + toNode.x;
        y = this.scene.centerY + toNode.y;
        [ex, ey] = this.getPortPosition("top", x, y);
        return {
          start: [cx, cy],
          end: [ex, ey],
          id: link.id,
        };
      });
      if (this.draggingLink) {
        let x, y, cy, cx;
        const fromNode = this.findNodeWithID(this.draggingLink.from);
        x = this.scene.centerX + fromNode.x;
        y = this.scene.centerY + fromNode.y;
        [cx, cy] = this.getPortPosition("bottom", x, y);
        // push temp dragging link, mouse cursor postion = link end postion
        lines.push({
          start: [cx, cy],
          end: [this.draggingLink.mx, this.draggingLink.my],
        });
      }
      return lines;
    },
  },
  mounted() {
    this.rootDivOffset.top = this.$el ? this.$el.offsetTop : 0;
    this.rootDivOffset.left = this.$el ? this.$el.offsetLeft : 0;
    // console.log(22222, this.rootDivOffset);
  },
  methods: {
    round(num, places) {
      if (!("" + num).includes("e")) {
        return +(Math.round(num + "e+" + places) + "e-" + places);
      } else {
        let arr = ("" + num).split("e");
        let sig = ""
        if (+arr[1] + places > 0) {
          sig = "+";
        }

        return +(Math.round(+arr[0] + "e" + sig + (+arr[1] + places)) + "e-" + places);
      }
    },
    aumenta() {
      if (this.scale < 1.9) {
        var number = this.scale + 0.1;
        number = this.round(number, 1);
        this.scale = number;
        switch (number) {
          case 1.9:
            this.marginLeftStyle = '2250px';
            this.marginTopStyle = '2250px';
            break;
          case 1.8:
            this.marginLeftStyle = '2000px';
            this.marginTopStyle = '2000px';
            break;
          case 1.7:
            this.marginLeftStyle = '1750px';
            this.marginTopStyle = '1750px';
            break;
          case 1.6:
            this.marginLeftStyle = '1500px';
            this.marginTopStyle = '1500px';
            break;
          case 1.5:
            this.marginLeftStyle = '1250px';
            this.marginTopStyle = '1250px';
            break;
          case 1.4:
            this.marginLeftStyle = '1000px';
            this.marginTopStyle = '1000px';
            break;
          case 1.3:
            this.marginLeftStyle = '750px';
            this.marginTopStyle = '750px';
            break;
          case 1.2:
            this.marginLeftStyle = '500px';
            this.marginTopStyle = '500px';
            break;
          case 1.1:
            this.marginLeftStyle = '250px';
            this.marginTopStyle = '250px';
            break;
          case 1.0:
            this.marginLeftStyle = '0px';
            this.marginTopStyle = '0px';
            break;
          case 0.9:
            this.marginLeftStyle = '-250px';
            this.marginTopStyle = '-250px';
            break;
          case 0.8:
            this.marginLeftStyle = '-500px';
            this.marginTopStyle = '-500px';
            break;
          case 0.7:
            this.marginLeftStyle = '-750px';
            this.marginTopStyle = '-750px';
            break;
          case 0.6:
            this.marginLeftStyle = '-1000px';
            this.marginTopStyle = '-1000px';
            break;
          case 0.5:
            this.marginLeftStyle = '-1250px';
            this.marginTopStyle = '-1250px';
            break;
          case 0.4:
            this.marginLeftStyle = '-1500px';
            this.marginTopStyle = '-1500px';
            break;
          case 0.3:
            this.marginLeftStyle = '-1750px';
            this.marginTopStyle = '-1750px';
            break;
        }
      }
    },
    diminui() {
      if (this.scale > 0.3) {
        var number = this.scale - 0.1;
        number = this.round(number, 1);
        this.scale = number;
        switch (number) {
          case 1.9:
            this.marginLeftStyle = '2250px';
            this.marginTopStyle = '2250px';
            break;
          case 1.8:
            this.marginLeftStyle = '2000px';
            this.marginTopStyle = '2000px';
            break;
          case 1.7:
            this.marginLeftStyle = '1750px';
            this.marginTopStyle = '1750px';
            break;
          case 1.6:
            this.marginLeftStyle = '1500px';
            this.marginTopStyle = '1500px';
            break;
          case 1.5:
            this.marginLeftStyle = '1250px';
            this.marginTopStyle = '1250px';
            break;
          case 1.4:
            this.marginLeftStyle = '1000px';
            this.marginTopStyle = '1000px';
            break;
          case 1.3:
            this.marginLeftStyle = '750px';
            this.marginTopStyle = '750px';
            break;
          case 1.2:
            this.marginLeftStyle = '500px';
            this.marginTopStyle = '500px';
            break;
          case 1.1:
            this.marginLeftStyle = '250px';
            this.marginTopStyle = '250px';
            break;
          case 1.0:
            this.marginLeftStyle = '0px';
            this.marginTopStyle = '0px';
            break;
          case 0.9:
            this.marginLeftStyle = '-250px';
            this.marginTopStyle = '-250px';
            break;
          case 0.8:
            this.marginLeftStyle = '-500px';
            this.marginTopStyle = '-500px';
            break;
          case 0.7:
            this.marginLeftStyle = '-750px';
            this.marginTopStyle = '-750px';
            break;
          case 0.6:
            this.marginLeftStyle = '-1000px';
            this.marginTopStyle = '-1000px';
            break;
          case 0.5:
            this.marginLeftStyle = '-1250px';
            this.marginTopStyle = '-1250px';
            break;
          case 0.4:
            this.marginLeftStyle = '-1500px';
            this.marginTopStyle = '-1500px';
            break;
          case 0.3:
            this.marginLeftStyle = '-1750px';
            this.marginTopStyle = '-1750px';
            break;
        }
      }
    },
    findNodeWithID(id) {
      return this.scene.nodes.find((item) => {
        return id === item.id;
      });
    },
    getPortPosition(type, x, y) {
      if (type === "top") {
        return [x + 40, y];
      } else if (type === "bottom") {
        return [x + 40, y + 80];
      }
    },
    linkingStart(index) {
      this.action.linking = true;
      this.draggingLink = {
        from: index,
        mx: 0,
        my: 0,
      };
    },
    linkingStop(index) {
      // add new Link
      if (this.draggingLink && this.draggingLink.from !== index) {
        // check link existence
        const existed = this.scene.links.find((link) => {
          return link.from === this.draggingLink.from && link.to === index;
        });
        if (!existed) {
          let maxID = Math.max(
            0,
            ...this.scene.links.map((link) => {
              return link.id;
            })
          );
          const newLink = {
            id: maxID + 1,
            from: this.draggingLink.from,
            to: index,
          };
          this.scene.links.push(newLink);
          this.$emit("linkAdded", newLink);
        }
      }
      this.draggingLink = null;
    },
    linkDelete(id) {
      const deletedLink = this.scene.links.find((item) => {
        return item.id === id;
      });
      if (deletedLink) {
        this.scene.links = this.scene.links.filter((item) => {
          return item.id !== id;
        });
        this.$emit("linkBreak", deletedLink);
      }
    },
    nodeSelected(id, e) {
      this.action.dragging = id;
      this.action.selected = id;
      this.$emit("nodeClick", id);
      this.mouse.lastX =
        e.pageX || e.clientX + document.documentElement.scrollLeft;
      this.mouse.lastY =
        e.pageY || e.clientY + document.documentElement.scrollTop;
    },
    handleMove(e) {
      if (this.action.linking) {
        [this.mouse.x, this.mouse.y] = getMousePosition(this.$el, e);
        [this.draggingLink.mx, this.draggingLink.my] = [
          this.mouse.x,
          this.mouse.y,
        ];
      }
      if (this.action.dragging) {
        this.mouse.x =
          e.pageX || e.clientX + document.documentElement.scrollLeft;
        this.mouse.y =
          e.pageY || e.clientY + document.documentElement.scrollTop;
        let diffX = this.mouse.x - this.mouse.lastX;
        let diffY = this.mouse.y - this.mouse.lastY;

        this.mouse.lastX = this.mouse.x;
        this.mouse.lastY = this.mouse.y;
        this.moveSelectedNode(diffX, diffY);
      }
      if (this.action.scrolling) {
        [this.mouse.x, this.mouse.y] = getMousePosition(this.$el, e);
        let diffX = this.mouse.x - this.mouse.lastX;
        let diffY = this.mouse.y - this.mouse.lastY;

        this.mouse.lastX = this.mouse.x;
        this.mouse.lastY = this.mouse.y;

        this.scene.centerX += diffX;
        this.scene.centerY += diffY;

        // this.hasDragged = true
      }
    },
    handleUp(e) {
      const target = e.target || e.srcElement;
      if (this.$el.contains(target)) {
        if (
          typeof target.className !== "string" ||
          target.className.indexOf("node-input") < 0
        ) {
          this.draggingLink = null;
        }
        if (
          typeof target.className === "string" &&
          target.className.indexOf("node-delete") > -1
        ) {
          // console.log('delete2', this.action.dragging);
          this.nodeDelete(this.action.dragging);
        }
      }
      this.action.linking = false;
      this.action.dragging = null;
      this.action.scrolling = false;
    },
    handleDown(e) {
      const target = e.target || e.srcElement;
      // console.log('for scroll', target, e.keyCode, e.which)
      if (
        (target === this.$el || target.matches("svg, svg *")) &&
        e.which === 1
      ) {
        this.action.scrolling = true;
        [this.mouse.lastX, this.mouse.lastY] = getMousePosition(this.$el, e);
        this.action.selected = null; // deselectAll
      }
      this.$emit("canvasClick", e);
    },
    moveSelectedNode(dx, dy) {
      let index = this.scene.nodes.findIndex((item) => {
        return item.id === this.action.dragging;
      });
      let left = this.scene.nodes[index].x + dx / this.scene.scale;
      let top = this.scene.nodes[index].y + dy / this.scene.scale;
      this.$set(
        this.scene.nodes,
        index,
        Object.assign(this.scene.nodes[index], {
          x: left,
          y: top,
        })
      );
    },
    nodeDelete(id) {
      this.scene.nodes = this.scene.nodes.filter((node) => {
        return node.id !== id;
      });
      this.scene.links = this.scene.links.filter((link) => {
        return link.from !== id && link.to !== id;
      });
      this.$emit("nodeDelete", id);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.button{
    display: flex;
    flex-grow: 0;
    align-items: center;
    justify-content: center;
    gap: 0.25rem;
    white-space: nowrap;
    border-width: 1px;
    padding: 0.5rem;
    font-weight: 600;
    --tw-text-opacity: 1;
    color: rgba(255, 255, 255, var(--tw-text-opacity));
    --tw-bg-opacity: 1;
    background-color: rgba(59, 130, 246, var(--tw-bg-opacity));
    --tw-border-opacity: 1;
    border-color: rgba(59, 130, 246, var(--tw-border-opacity));
    height: 1.88rem;
    line-height: inherit;
    cursor: pointer;
    text-transform: none;
    font-family: inherit;
    font-size: 100%;
    margin: 0;
}
.button:hover{
    --tw-text-opacity: 1;
    color: rgba(29, 78, 216, var(--tw-text-opacity));
    --tw-bg-opacity: 1;
    background-color: rgba(219, 234, 254, var(--tw-bg-opacity));
    transition-property: background-color, border-color, color, fill, stroke, opacity, box-shadow, transform, filter, backdrop-filter, -webkit-backdrop-filter;
    transition-duration: 300ms;
    transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}
.direita{

  border-top-right-radius: 5px;
    border-bottom-right-radius: 5px;
}
.esquerda{
  
  border-top-left-radius: 5px;
    border-bottom-left-radius: 5px;
}
.flowchart-container {
  width: 100%;
  margin: 0;
  background: radial-gradient(#8dc3f9, white);
  -webkit-background-image: -webkit-repeating-radial-gradient(#8dc3f9, white);
  -moz-background-image: -moz-radial-gradient(#8dc3f9, white);
  -o-background-image: -o-radial-gradient(#8dc3f9, white);
  background-size: 5px 5px;
  -webkit-background-size: 4px 4px;
  -moz-background-size: 4px 4px;
  -o-background-size: 4px 4px;
  position: relative;
  overflow: hidden;

  svg {
    cursor: grab;
  }
}
</style>
