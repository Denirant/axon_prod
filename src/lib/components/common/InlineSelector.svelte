<!-- InlineSelector.svelte -->
<script>
    import { tweened } from 'svelte/motion';
    import { cubicOut } from 'svelte/easing';
    import { createEventDispatcher } from 'svelte';
  
    // Create event dispatcher to notify parent components
    const dispatch = createEventDispatcher();
  
    // Props
    /** @type {Array<{id: string | number, label: string}>} */
    export let options = [];
    
    /** @type {string | number} */
    export let value = null;
    
    /** @type {Function} */
    export let onChange = () => {};
    
    /** @type {boolean} */
    export let disabled = false;
    
    /** @type {string} */
    export let name = '';
  
    // State for the active indicator's position and dimensions
    const indicatorPosition = tweened({ x: 0, width: 0 }, {
      duration: 200,
      easing: cubicOut
    });
  
    // DOM nodes references
    let itemsRef = [];
    let containerRef;
  
    // Update indicator position when selected value changes or on resize
    $: if (containerRef && itemsRef.length > 0) {
      const selectedIndex = options.findIndex(option => option.id === value);
      if (selectedIndex >= 0 && itemsRef[selectedIndex]) {
        const item = itemsRef[selectedIndex];
        const containerRect = containerRef.getBoundingClientRect();
        const itemRect = item.getBoundingClientRect();
        
        indicatorPosition.set({
          x: itemRect.left - containerRect.left - 5,
          width: itemRect.width
        });
      }
    }
  
    function handleClick(option) {
      if (disabled) return;
      value = option.id; // Update value locally for reactivity
      onChange(option.id);
      dispatch('change', { value: option.id }); // Dispatch event to parent
    }
  
    function handleKeydown(e, option) {
      if (disabled) return;
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        value = option.id; // Update value locally for reactivity
        onChange(option.id);
        dispatch('change', { value: option.id }); // Dispatch event to parent
      }
    }
  </script>
  
  <div
    class="relative inline-flex border border-gray-100 bg-gray-50 rounded-2xl p-1 gap-1 select-none"
    class:opacity-60={disabled}
    class:cursor-not-allowed={disabled}
    bind:this={containerRef}
    role="radiogroup"
    aria-labelledby={name ? `${name}-label` : undefined}
  >
    <div 
      class="absolute h-[calc(100%-8px)] bg-gray-700 rounded-xl z-0 top-1 pointer-events-none"
      style="transform: translateX({$indicatorPosition.x}px); width: {$indicatorPosition.width}px;"
    ></div>
    
    {#each options as option, i}
      <button
        type="button"
        class="relative z-10 px-3 py-1 bg-transparent rounded-xl border-none text-sm font-normal cursor-pointer whitespace-nowrap outline-none transition-colors duration-50"
        class:text-white={option.id === value}
        class:text-gray-500={option.id !== value}
        class:hover:text-gray-600={option.id !== value && !disabled}
        class:hover:bg-gray-100={option.id !== value && !disabled}
        class:cursor-not-allowed={disabled}
        bind:this={itemsRef[i]}
        role="radio"
        aria-checked={option.id === value}
        disabled={disabled}
        on:click={() => handleClick(option)}
        on:keydown={(e) => handleKeydown(e, option)}
        tabindex={option.id === value || (i === 0 && value === null) ? 0 : -1}
      >
        {option.label}
      </button>
    {/each}
  </div>