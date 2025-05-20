<!-- Script JS -->

<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import monumentos from "/src/data/Monumentos - Hoja 1.csv";

  let svgMap = {};

  // Convierte los valores altura, anio y visitas a números
  monumentos.forEach(m => {
    m.altura = +m.altura;
    m.anio = +m.anio;
    m.visitas = +m.visitas;
  });

  // Crea una escala de raíz cuadrada (D3.js) que transforma alturas reales (entre 10 y 2430 metros) a un rango visual de tamaños (entre 30 y 100 píxeles).
  const escalaAltura = d3.scaleSqrt()
    .domain([10, 2430])
    .range([30, 100]);

  // Asigna un color específico a cada calificación ("Malo", "Regular", "Bueno", "Excelente").
  const colorCalificacion = d3.scaleOrdinal()
    .domain(["Malo", "Regular", "Bueno", "Excelente"])
    .range(["#912F27", "#EA7B4D", "#A2D4D3", "#3B7B78"]);

  // Lista fija de continentes para estructurar la matriz visual (eje Y).
  const continentes = ["África", "América", "Asia", "Europa", "Oceanía"];

  // Genera los siglos del milenio (del 1000 al 1900, eje X) para posicionar los monumentos.
  const siglos = Array.from({ length: 10 }, (_, i) => 1000 + i * 100);

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
  onMount(fetchSVGs);

  let tooltip;

  function showTooltip(event, m) {
    tooltip.innerHTML = `
      <strong>${m.nombre}</strong><br>
      Continente: ${m.continente}<br>
      Altura: ${m.altura} m<br>
      Año: ${m.anio}<br>
      Visitas: ${m.visitas} M<br>
      Calificación: ${m.calificacion}
    `;
    tooltip.style.display = "block";
    tooltip.style.left = event.pageX + 15 + "px";
    tooltip.style.top = event.pageY + 15 + "px";
  }

  function hideTooltip() {
    tooltip.style.display = "none";
  }

</script>

<main>
  <h1>Monumentos icónicos del mundo</h1>
  <h2>Una mirada visual a los monumentos más representativos de la historia global. Cada celda representa un siglo. Cada fila, un continente. Cada figura, un monumento. Todo en una sola vista.</h2>
  <div class="sistema">
    <div class = "calificacion"> <strong>Calificación:</strong>
      <div class="colores">
        <div class="color">
          <div class="cuadro" style="background:#912F27">Malo</div>
          <span>Menos de 1 millon</span>
        </div>
        <div class="color">
          <div class="cuadro" style="background:#EA7B4D">Regular</div>
          <span>1 millon - 2 millones</span>
        </div>
        <div class="color">
          <div class="cuadro" style="background:#A2D4D3">Bueno</div>
          <span>2 millones - 5 millon</span>
        </div>
        <div class="color">
          <div class="cuadro" style="background:#3B7B78">Excelente</div>
          <span>Más de 5 millones</span>
        </div>
      </div>
    </div>

    <div class = "visitas">
        <strong>Visitas anuales:</strong>
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
  </div>

  <div class="matriz">
    {#each continentes as cont}
      <div class="fila">
        <div class="label-y">{cont}</div>
        <div class="celdas">
          {#each siglos as siglo}
            <div class="celda">
              {#each monumentos.filter(m => m.continente === cont && m.anio >= siglo && m.anio < siglo + 100) as m}
                <div
                  class="monumento"
                  style="width: {escalaAltura(m.altura)}px; height: {escalaAltura(m.altura)}px; color: {colorCalificacion(m.calificacion)}"
                  on:mousemove={(e) => showTooltip(e, m)}
                  on:mouseleave={hideTooltip}
                  role="img"
                  aria-label={`Monumento: ${m.nombre}, ${m.calificacion}, ${m.visitas}M visitas`}
                >
                  {@html svgMap[svgPorVisitas(m.visitas)] || ''}
                </div>
              {/each}
            </div>
          {/each}
        </div>
      </div>
    {/each}

    <div class="fila">
      <div class="label-y"></div>
      {#each siglos as siglo}
        <div class="label-x">{siglo}s</div>
      {/each}
    </div>
  </div>

  <div class="tooltip" bind:this={tooltip}></div>

</main>
 
