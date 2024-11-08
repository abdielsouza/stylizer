<script lang="ts">
    import { onMount } from 'svelte';
  
    let selectedColor = '#ff0000';
    let selectedFont = 'Arial';
    let fontSize = '16';
    let fontWeight = 'normal';
    let fontStyle = 'normal';
    let generatedCSS = '';
  
    let hue = 0;
    let saturation = 100;
    let lightness = 50;
  
    let isSelectingMain = false;
    let isSelectingHue = false;
  
    let isDarkMode = false;
  
    const fonts = [
      'Arial', 'Helvetica', 'Times New Roman', 'Courier', 'Verdana',
      'Georgia', 'Palatino', 'Garamond', 'Bookman', 'Comic Sans MS',
      'Trebuchet MS', 'Arial Black', 'Impact'
    ];
  
    function startMainSelection(event: any) {
      isSelectingMain = true;
      updateMainSelection(event);
    }
  
    function updateMainSelection(event: any) {
      if (isSelectingMain) {
        const rect = event.target.getBoundingClientRect();
        saturation = Math.round((event.clientX - rect.left) / rect.width * 100);
        lightness = 100 - Math.round((event.clientY - rect.top) / rect.height * 100);
        updateSelectedColor();
      }
    }
  
    function endMainSelection() {
      isSelectingMain = false;
    }
  
    function startHueSelection(event: any) {
      isSelectingHue = true;
      updateHueSelection(event);
    }
  
    function updateHueSelection(event: any) {
      if (isSelectingHue) {
        const rect = event.target.getBoundingClientRect();
        hue = Math.round((event.clientY - rect.top) / rect.height * 360);
        updateSelectedColor();
      }
    }
  
    function endHueSelection() {
      isSelectingHue = false;
    }
  
    function updateSelectedColor() {
      selectedColor = hslToHex(hue, saturation, lightness);
      updateCSS();
    }
  
    function hslToHex(h: number, s: number, l: number) {
      l /= 100;
      const a = s * Math.min(l, 1 - l) / 100;
      const f = (n: number) => {
        const k = (n + h / 30) % 12;
        const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
        return Math.round(255 * color).toString(16).padStart(2, '0');
      };
      return `#${f(0)}${f(8)}${f(4)}`;
    }
  
    function updateCSS() {
      generatedCSS = `
  font-family: ${selectedFont}, sans-serif;
  font-size: ${fontSize}px;
  font-weight: ${fontWeight};
  font-style: ${fontStyle};
  color: ${selectedColor};
      `.trim();
    }
  
    function copyToClipboard() {
      navigator.clipboard.writeText(generatedCSS);
      alert('CSS copiado para a área de transferência!');
    }
  
    function toggleDarkMode() {
      isDarkMode = !isDarkMode;
      document.body.classList.toggle('dark', isDarkMode);
    }
  
    onMount(() => {
      updateCSS();
    });
  </script>
  
  <main class="container mx-auto p-4 transition-colors duration-300" class:dark={isDarkMode}>
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl dark:text-white font-bold">Estilizador de Websites</h1>
      <button
        on:click={toggleDarkMode}
        class="px-4 py-2 rounded-full bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200 transition-colors duration-300"
      >
        {isDarkMode ? '☀️ Modo Claro' : '🌙 Modo Escuro'}
      </button>
    </div>
  
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-md transition-colors duration-300">
        <h2 class="text-xl font-semibold mb-4 text-gray-800 dark:text-gray-200">Seletor de Cores</h2>
        <div class="flex space-x-4">
          <div class="relative flex-grow">
            <div 
              class="w-full h-48 rounded-lg cursor-crosshair"
              style="background: linear-gradient(to bottom, transparent, #000), linear-gradient(to right, #fff, hsl({hue}, 100%, 50%)); background-size: 100% 100%, 100% 100%;"
              on:mousedown={startMainSelection}
              on:mousemove={updateMainSelection}
              on:mouseup={endMainSelection}
              on:mouseleave={endMainSelection}
            ></div>
            <div 
              class="absolute w-4 h-4 rounded-full border-2 border-white shadow-md pointer-events-none"
              style="left: {saturation}%; top: {100 - lightness}%; transform: translate(-50%, -50%);"
            ></div>
          </div>
          <div class="relative w-6">
            <div 
              class="w-full h-48 rounded-lg cursor-pointer"
              style="background: linear-gradient(to bottom, #f00 0%, #ff0 17%, #0f0 33%, #0ff 50%, #00f 67%, #f0f 83%, #f00 100%);"
              on:mousedown={startHueSelection}
              on:mousemove={updateHueSelection}
              on:mouseup={endHueSelection}
              on:mouseleave={endHueSelection}
            ></div>
            <div 
              class="absolute w-6 h-2 bg-white border border-gray-300 shadow-md pointer-events-none"
              style="top: {hue / 360 * 100}%; transform: translateY(-50%);"
            ></div>
          </div>
        </div>
        <div class="flex items-center space-x-4 mt-4">
          <div 
            class="w-12 h-12 rounded-lg shadow-inner"
            style="background-color: {selectedColor};"
          ></div>
          <input
            type="text"
            bind:value={selectedColor}
            on:input={updateCSS}
            class="border rounded px-2 py-1 flex-grow bg-white dark:bg-gray-700 text-gray-800 dark:text-gray-200 transition-colors duration-300"
          />
        </div>
      </div>
  
      <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-md transition-colors duration-300">
        <h2 class="text-xl font-semibold mb-4 text-gray-800 dark:text-gray-200">Estilo de Fonte</h2>
        <div class="space-y-4">
          <div>
            <label for="font-select" class="block mb-1 text-gray-800 dark:text-gray-200">Fonte:</label>
            <select
              id="font-select"
              bind:value={selectedFont}
              on:change={updateCSS}
              class="w-full border rounded px-2 py-1 bg-white dark:bg-gray-700 text-gray-800 dark:text-gray-200 transition-colors duration-300"
            >
              {#each fonts as font}
                <option value={font}>{font}</option>
              {/each}
            </select>
          </div>
          <div>
            <label for="font-size" class="block mb-1 text-gray-800 dark:text-gray-200">Tamanho da Fonte:</label>
            <input
              id="font-size"
              type="number"
              bind:value={fontSize}
              on:input={updateCSS}
              min="8"
              max="72"
              class="w-full border rounded px-2 py-1 bg-white dark:bg-gray-700 text-gray-800 dark:text-gray-200 transition-colors duration-300"
            />
          </div>
          <div>
            <label for="font-weight" class="block mb-1 text-gray-800 dark:text-gray-200">Peso da Fonte:</label>
            <select
              id="font-weight"
              bind:value={fontWeight}
              on:change={updateCSS}
              class="w-full border rounded px-2 py-1 bg-white dark:bg-gray-700 text-gray-800 dark:text-gray-200 transition-colors duration-300"
            >
              <option value="normal">Normal</option>
              <option value="bold">Negrito</option>
            </select>
          </div>
          <div>
            <label for="font-style" class="block mb-1 text-gray-800 dark:text-gray-200">Estilo da Fonte:</label>
            <select
              id="font-style"
              bind:value={fontStyle}
              on:change={updateCSS}
              class="w-full border rounded px-2 py-1 bg-white dark:bg-gray-700 text-gray-800 dark:text-gray-200 transition-colors duration-300"
            >
              <option value="normal">Normal</option>
              <option value="italic">Itálico</option>
            </select>
          </div>
        </div>
      </div>
    </div>
  
    <div class="mt-8 bg-white dark:bg-gray-800 p-6 rounded-lg shadow-md transition-colors duration-300">
      <h2 class="text-xl font-semibold mb-4 text-gray-800 dark:text-gray-200">CSS Gerado</h2>
      <pre class="bg-gray-100 dark:bg-gray-700 p-4 rounded text-gray-800 dark:text-gray-200 transition-colors duration-300">{generatedCSS}</pre>
      <button
        on:click={copyToClipboard}
        class="mt-4 bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 transition-colors duration-300"
      >
        Copiar CSS
      </button>
    </div>
  
    <div class="mt-8 bg-white dark:bg-gray-800 p-6 rounded-lg shadow-md transition-colors duration-300">
      <h2 class="text-xl font-semibold mb-4 text-gray-800 dark:text-gray-200">Prévia</h2>
      <p style={generatedCSS} class="text-gray-800 dark:text-gray-200 transition-colors duration-300">
        Este é um exemplo de texto com o estilo selecionado.
      </p>
    </div>
  </main>
  
  <style>
    :global(body) {
      background-color: #f0f0f0;
      transition: background-color 0.3s ease;
      color: black;
    }
  
    :global(body.dark) {
      background-color: #1a202c;
    }
  </style>