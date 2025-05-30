<script lang="ts">
  import { clickOutside } from '$lib/actions/click-outside';
  import { focusTrap } from '$lib/actions/focus-trap';
  import ModalHeader from '$lib/components/shared-components/modal-header.svelte';
  import { generateId } from '$lib/utils/generate-id';
  import type { Snippet } from 'svelte';
  import { fade } from 'svelte/transition';

  interface Props {
    onClose: () => void;
    title: string;
    /**
     * If true, the logo will be displayed next to the modal title.
     */
    showLogo?: boolean;
    /**
     * Optional icon to display next to the modal title, if `showLogo` is false.
     */
    icon?: string | undefined;
    /**
     * Sets the width of the modal.
     *
     * - `wide`: 48rem
     * - `narrow`: 28rem
     * - `auto`: fits the width of the modal content, up to a maximum of 32rem
     */
    width?: 'extra-wide' | 'wide' | 'narrow' | 'auto';
    stickyBottom?: Snippet;
    children?: Snippet;
  }

  let {
    onClose,
    title,
    showLogo = false,
    icon = undefined,
    width = 'narrow',
    stickyBottom,
    children,
  }: Props = $props();

  /**
   * Unique identifier for the modal.
   */
  let id: string = generateId();

  let titleId = $derived(`${id}-title`);
  let isStickyBottom = $derived(!!stickyBottom);

  let modalWidth = $state<string>();

  $effect(() => {
    switch (width) {
      case 'extra-wide': {
        modalWidth = 'w-[56rem]';
        break;
      }

      case 'wide': {
        modalWidth = 'w-[48rem]';
        break;
      }

      case 'narrow': {
        modalWidth = 'w-[28rem]';
        break;
      }

      default: {
        modalWidth = 'sm:max-w-4xl';
      }
    }
  });
</script>

<section
  role="presentation"
  in:fade={{ duration: 100 }}
  out:fade={{ duration: 100 }}
  class="fixed start-0 top-0 flex h-dvh w-dvw place-content-center place-items-center bg-black/40"
  onkeydown={(event) => {
    event.stopPropagation();
  }}
  use:focusTrap
>
  <div
    class="flex flex-col max-h-[min(95dvh,60rem)] max-w-[95vw] {modalWidth} overflow-hidden rounded-3xl bg-immich-bg shadow-md dark:bg-immich-dark-gray dark:text-immich-dark-fg pt-3 pb-4"
    use:clickOutside={{ onOutclick: onClose, onEscape: onClose }}
    tabindex="-1"
    aria-modal="true"
    aria-labelledby={titleId}
  >
    <div class="immich-scrollbar overflow-y-auto pt-1" class:pb-4={isStickyBottom}>
      <ModalHeader id={titleId} {title} {showLogo} {icon} {onClose} />
      <div class="px-5 pt-0 mb-5">
        {@render children?.()}
      </div>
    </div>
    {#if isStickyBottom}
      <div
        class="flex flex-col sm:flex-row justify-end w-full gap-2 sm:gap-4 sticky pt-4 px-5 bg-immich-bg dark:bg-immich-dark-gray border-t border-gray-200 dark:border-gray-500"
      >
        {@render stickyBottom?.()}
      </div>
    {/if}
  </div>
</section>
