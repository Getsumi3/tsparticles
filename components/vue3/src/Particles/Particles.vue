<template>
  <div :id="id"></div>
</template>

<script lang="ts">
import { nextTick, PropType } from "vue";
import { Options, Vue } from "vue-class-component";
import { tsParticles } from "tsparticles";
import type { Container, ISourceOptions, Main } from "tsparticles";

export type IParticlesProps = ISourceOptions;
export type IParticlesParams = IParticlesProps;

@Options({
  props: {
    id: {
      type: String,
      required: true
    },
    options: {
      type: Object as PropType<IParticlesProps>
    },
    url: {
      type: String
    },
    particlesLoaded: {
      type: Function as PropType<(container: Container) => void>
    },
    particlesInit: {
      type: Function as PropType<(tsParticles: Main) => void>
    }
  }
})
export default class Particles extends Vue {
  private id!: string;
  private options?: IParticlesProps;
  private url?: string;
  private particlesLoaded?: (container: Container) => void;
  private particlesInit?: (tsParticles: Main) => void;
  private container?: Container;

  public mounted(): void {
    nextTick(() => {
      if (!this.id) {
        throw new Error("Prop 'id' is required!");
      }

      tsParticles.init();

      if (this.particlesInit) {
        this.particlesInit(tsParticles);
      }

      const cb = (container?: Container) => {
        this.container = container;

        if (this.particlesLoaded && container) {
          this.particlesLoaded(container);
        }
      };

      if (this.url) {
        tsParticles
            .loadJSON(this.id, this.url)
            .then(cb);
      } else {
        tsParticles
            .load(this.id, this.options ?? {})
            .then(cb);
      }
    });
  }

  public beforeDestroy(): void {
    if (this.container) {
      this.container.destroy();
    }
  }
}
</script>
