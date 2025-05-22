<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import monumentos from "/src/data/Monumentos - Hoja 1.csv";

  let svgMap = {};
  let activeColumn = null;

  // Convierte los valores altura, anio y visitas a números
  monumentos.forEach(m => {
    m.altura = +m.altura;
    m.anio = +m.anio;
    m.visitas = +m.visitas;
  });

  // Asigna un color específico a cada calificación ("Malo", "Regular", "Bueno", "Excelente").
  const colorCalificacion = d3.scaleOrdinal()
    .domain(["Malo", "Regular", "Bueno", "Excelente"])
    .range(["#912F27", "#EA7B4D", "#A2D4D3", "#3B7B78"]);

  // Lista fija de continentes para estructurar la matriz visual (eje Y).
  const continentes = ["África", "América", "Asia", "Europa", "Oceanía"];

  // Genera los siglos del milenio (del 1000 al 1900, eje X) para posicionar los monumentos.
  const siglos = Array.from({ length: 10 }, (_, i) => 1000 + i * 100);
  
  // Función para convertir siglos a números romanos
  function sigloARomano(siglo) {
    const sigloNumero = siglo / 100;
    const romanos = ["X", "XI", "XII", "XIII", "XIV", "XV", "XVI", "XVII", "XVIII", "XIX"];
    return romanos[sigloNumero - 10]; // Restamos 10 porque empezamos en el siglo X (1000s)
  }

  // Devuelve el nombre del archivo SVG correspondiente según la cantidad de visitas anuales.
  function svgPorVisitas(visitas) {
    if (visitas < 1) return "menosde1mm.svg";
    if (visitas < 1.6) return "1mm1.6mm.svg";
    if (visitas < 2) return "1.6mm2mm.svg";
    if (visitas < 2.6) return "2mm2.6mm.svg";
    if (visitas < 3.5) return "2.6mm3.5mm.svg";
    if (visitas < 5) return "3.5mm5mm.svg";
    if (visitas < 7) return "5mm7mm.svg";
    if (visitas < 9) return "7mm8mm.svg";
    return "masde9mm.svg";
  }

  async function fetchSVGs() {
    const nombres = monumentos.map(m => svgPorVisitas(m.visitas));
    const unicos = Array.from(new Set(nombres));
    // Modifica el contenido del SVG para permitir cambiar el color con CSS
    for (const nombre of unicos) {
      const res = await fetch(`/images/${nombre}`);
      let raw = await res.text();
      raw = raw.replace(/fill="(?!none).*?"/g, 'fill="currentColor"');
      svgMap[nombre] = raw;
    }
  }

  // Hace que los SVGs se carguen automáticamente cuando la visualización aparece en pantalla.
  onMount(() => {
    fetchSVGs();
  });

  let tooltip;

  function showTooltip(event, m) {
    tooltip.innerHTML = `
      <div class="tooltip-header" style="background-color: ${colorCalificacion(m.calificacion)}">
        <strong>${m.nombre}</strong>
      </div>
      <div class="tooltip-content">
        <div class="tooltip-row"><span class="tooltip-label">Continente:</span> ${m.continente}</div>
        <div class="tooltip-row"><span class="tooltip-label">Altura:</span> ${m.altura.toLocaleString()} m</div>
        <div class="tooltip-row"><span class="tooltip-label">Año:</span> ${m.anio}</div>
        <div class="tooltip-row"><span class="tooltip-label">Visitas:</span> ${m.visitas} M</div>
        <div class="tooltip-row"><span class="tooltip-label">Calificación:</span> ${m.calificacion}</div>
      </div>
    `;
    tooltip.style.display = "block";
    tooltip.style.left = event.pageX + 15 + "px";
    tooltip.style.top = event.pageY + 15 + "px";
  }

  function hideTooltip() {
    tooltip.style.display = "none";
  }

  function showSigloTooltip(event, siglo) {
    tooltip.innerHTML = `
      <div class="tooltip-header tooltip-header-siglo">
        <strong>Siglo ${sigloARomano(siglo)}</strong>
      </div>
      <div class="tooltip-content">
        <div class="tooltip-row">Años ${siglo} - ${siglo + 99}</div>
      </div>
    `;
    tooltip.style.display = "block";
    tooltip.style.left = event.pageX + 15 + "px";
    tooltip.style.top = event.pageY + 15 + "px";
  }

  function setActiveColumn(siglo) {
    activeColumn = siglo;
  }

  function clearActiveColumn() {
    activeColumn = null;
  }

  // Función para obtener monumentos por celda (limitado a 9)
  function getMonumentosPorCelda(continente, siglo) {
    return monumentos
      .filter(m => m.continente === continente && m.anio >= siglo && m.anio < siglo + 100)
      .slice(0, 9); // Limitar a máximo 9 monumentos por celda
  }
</script>

<main>
  <div class="hero-section">
    <div class="hero-content">
      <h1>Monumentos icónicos del mundo</h1>
      <h2>Una mirada visual a los monumentos más representativos de la historia global. Cada celda representa un siglo. Cada fila, un continente. Cada figura, un monumento. Todo en una sola vista.</h2>
    </div>
  </div>
  
  <div class="sistema">
    <div class="leyenda-container">
      <div class="visitas">
        <h3>Visitas anuales</h3>
        <div class="formas">
          <div class="forma">
            <img src="/images/menosde1mm.svg" alt="Menos de 1 millón de visitas anuales"/>
            <span>Menos de 1M</span>
          </div>
          <div class="forma">
            <img src="/images/1mm1.6mm.svg" alt="Entre 1 millón y 1.6 millones de visitas anuales"/>
            <span>1M – 1.6M</span>
          </div>
          <div class="forma">
            <img src="/images/1.6mm2mm.svg" alt="Entre 1.6 millones y 2 millones de visitas anuales"/>
            <span>1.6M – 2M</span>
          </div>
          <div class="forma">
            <img src="/images/2mm2.6mm.svg" alt="Entre 2 millones y 2.6 millones de visitas anuales"/>
            <span>2M – 2.6M</span>
          </div>
          <div class="forma">
            <img src="/images/2.6mm3.5mm.svg" alt="Entre 2.6 millones y 3.5 millones de visitas anuales"/>
            <span>2.6M – 3.5M</span>
          </div>
          <div class="forma">
            <img src="/images/3.5mm5mm.svg" alt="Entre 3.5 millones y 5 millones de visitas anuales"/>
            <span>3.5M – 5M</span>
          </div>
          <div class="forma">
            <img src="/images/5mm7mm.svg" alt="Entre 5 millones y 7 millones de visitas anuales"/>
            <span>5M – 7M</span>
          </div>
          <div class="forma">
            <img src="/images/7mm8mm.svg" alt="Entre 7 millones y 9 millones de visitas anuales"/>
            <span>7M – 9M</span>
          </div>
          <div class="forma">
            <img src="/images/masde9mm.svg" alt="Más de 9 millones de visitas anuales"/>
            <span>Más de 9M</span>
          </div>
        </div>
      </div>

      <div class="calificacion"> 
        <h3>Calificación (medido en cantidad de visitas)</h3>
        <div class="colores">
          <div class="color">
            <div class="color-circle" style="background-color: #912F27;"></div>
            <span class="color-name">Malo</span>
            <span class="color-desc">Menos de 1M</span>
          </div>
          <div class="color">
            <div class="color-circle" style="background-color: #EA7B4D;"></div>
            <span class="color-name">Regular</span>
            <span class="color-desc">1M - 2M</span>
          </div>
          <div class="color">
            <div class="color-circle" style="background-color: #A2D4D3;"></div>
            <span class="color-name">Bueno</span>
            <span class="color-desc">2M - 5M</span>
          </div>
          <div class="color">
            <div class="color-circle" style="background-color: #3B7B78;"></div>
            <span class="color-name">Excelente</span>
            <span class="color-desc">Más de 5M</span>
          </div>
        </div>
      </div>
    </div>

    <div class="instrucciones">
      <p>Pase el cursor sobre un monumento para ver sus detalles. Pase el cursor sobre un siglo para resaltar toda la columna.</p>
    </div>
  </div>

  <div class="matriz-container">
    <!-- Fila de siglos en números romanos (arriba) -->
    <div class="fila fila-siglos">
      <div class="label-y-spacer"></div>
      <div class="celdas-header">
        {#each siglos as siglo}
          <div 
            class="siglo-label" 
            class:active={activeColumn === siglo}
            on:mouseenter={(e) => {
              setActiveColumn(siglo);
              showSigloTooltip(e, siglo);
            }}
            on:mouseleave={() => {
              clearActiveColumn();
              hideTooltip();
            }}
            role="button"
            tabindex="0"
            aria-label={`Siglo ${sigloARomano(siglo)}, años ${siglo} a ${siglo + 99}`}
          >
            {sigloARomano(siglo)}
          </div>
        {/each}
      </div>
    </div>
    
    <div class="matriz">
      {#each continentes as cont}
        <div class="fila">
          <div class="label-y">{cont}</div>
          <div class="celdas">
            {#each siglos as siglo}
              <div class="celda" class:active-celda={activeColumn === siglo}>
                <div class="grid-3x3">
                  {#each Array(9).fill(null) as _, index}
                    <div class="celda-pequena">
                      {#if getMonumentosPorCelda(cont, siglo)[index]}
                        {@const m = getMonumentosPorCelda(cont, siglo)[index]}
                        <div
                          class="monumento"
                          style="color: {colorCalificacion(m.calificacion)}"
                          on:mousemove={(e) => showTooltip(e, m)}
                          on:mouseleave={hideTooltip}
                          role="img"
                          aria-label={`Monumento: ${m.nombre}, ${m.calificacion}, ${m.visitas}M visitas`}
                        >
                          {@html svgMap[svgPorVisitas(m.visitas)] || ''}
                        </div>
                      {/if}
                    </div>
                  {/each}
                </div>
              </div>
            {/each}
          </div>
        </div>
      {/each}
    </div>
  </div>

  <footer>
    <div class="footer-content">
      <p>© {new Date().getFullYear()} - Visualización de monumentos icónicos del mundo</p>
    </div>
  </footer>

  <div class="tooltip" bind:this={tooltip}></div>

</main>