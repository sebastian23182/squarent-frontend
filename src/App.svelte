<script lang='ts'>
  import { type Property } from './types/property.d'
  import styles from './styles/property.module.css'

  initMap()

  let closeActive = false
  let createMarkers: (properties: Property[]) => void

  async function getProperties(region: string) {
    try {
      const response = await fetch(`https://squarent-backend-production.up.railway.app/fr/${region}`, { method: "POST" })
      if (!response.ok) throw new Error(`Error fetching properties for ${region}: ${response.statusText}`)
      const data = await response.json()
      createMarkers(data)
    } catch (error) {
      console.error(error)
    }
  }

  const VALLE_DE_ABURRA = {
    north: 6.405183,
    south: 6.062231, 
    west: -75.682015,
    east: -75.504432,
  }

  Object.freeze(VALLE_DE_ABURRA)
  
  async function initMap() {
    const { Map } = await google.maps.importLibrary('maps') as google.maps.MapsLibrary
    const { AdvancedMarkerElement } = await google.maps.importLibrary('marker') as google.maps.MarkerLibrary
    type Marker = InstanceType<typeof AdvancedMarkerElement>
    const parser = new DOMParser()

    const map = new Map(document.getElementById('map') as HTMLElement, {
      center: { lat: 6.3382, lng: -75.5586 },
      zoom: 17,
      minZoom: 17,
      maxZoom: 22,
      mapId: '4504f8b37365c3d0',
      restriction: {
        latLngBounds: VALLE_DE_ABURRA,
        strictBounds: true,
      },
      scrollwheel: true,
      disableDoubleClickZoom: true,     
      gestureHandling: 'auto',
      mapTypeId: google.maps.MapTypeId.HYBRID
    })

    const markerSVG = `<svg width='50' height='50' stroke='white' stroke-width='24' version='1.1' viewBox='0 0 1200 1200' xmlns='http://www.w3.org/2000/svg'><path d='m600.01 60c-252.02 0-456.35 204.32-456.35 456.37 0 142.68 76.246 258.24 167.94 353.64 66.348 69.074 288.41 269.99 288.41 269.99s240.2-217.62 261.65-239.05c101.09-101.15 194.68-230.66 194.68-384.57-0.011719-252.05-204.32-456.38-456.32-456.38zm184.79 462.95v228.88h-142.04v-169.57h-85.5v169.6l-142.12-0.003906v-228.9h-58.344l243.18-247.96 87.574 89.258v-45.562h68.102v114.97l87.59 89.293z'/></svg>`
    const baseIcon = parser.parseFromString(markerSVG, 'image/svg+xml').documentElement

    createMarkers = async function(properties: Property[]) {
      properties.forEach((property) => {
        const icon = baseIcon.cloneNode(true) as SVGElement
        icon.classList.add(styles.icon)
    
        const marker = new AdvancedMarkerElement({
          map,
          position: { lat: property.location_point[0], lng: property.location_point[1] },
          title: property.price.toString(),
          content: icon,
          gmpClickable: true,
        })

        marker.addListener('click', () => {
          if (!closeActive) toggleHighlight(marker, property, icon)
        })
      })
  }

    const REGIONS = ['bello', 'medellin', 'caldas', 'sabaneta', 'envigado', 'girardota', 'itagui', 'copacabana', 'estrella']
    Object.freeze(REGIONS)

    await Promise.all(REGIONS.map(REGION => getProperties(REGION)))

    function buildContent(property: Property, markerView: Marker, icon: SVGElement) {  
      const content = document.createElement('div')
      content.classList.add(styles.property)
      let position = 0
      const imageWidth = 254

      function redirect() {
        window.open(property.link, 'noopener,noreferrer')
      }

      content.innerHTML = `
        <div class=${styles.corner}></div>
        <div class=${styles.slider}>
          <button class=${styles.prev}>&#9664;</button>
          <button class=${styles.link} title="ir al sitio...">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="36" height="36"><path d="M18.3638 15.5355L16.9496 14.1213L18.3638 12.7071C20.3164 10.7545 20.3164 7.58866 18.3638 5.63604C16.4112 3.68341 13.2453 3.68341 11.2927 5.63604L9.87849 7.05025L8.46428 5.63604L9.87849 4.22182C12.6122 1.48815 17.0443 1.48815 19.778 4.22182C22.5117 6.95549 22.5117 11.3876 19.778 14.1213L18.3638 15.5355ZM15.5353 18.364L14.1211 19.7782C11.3875 22.5118 6.95531 22.5118 4.22164 19.7782C1.48797 17.0445 1.48797 12.6123 4.22164 9.87868L5.63585 8.46446L7.05007 9.87868L5.63585 11.2929C3.68323 13.2455 3.68323 16.4113 5.63585 18.364C7.58847 20.3166 10.7543 20.3166 12.7069 18.364L14.1211 16.9497L15.5353 18.364ZM14.8282 7.75736L16.2425 9.17157L9.17139 16.2426L7.75717 14.8284L14.8282 7.75736Z"></path></svg>
          </button>
          <div class=${styles.frame}>
            ${property.images.map((image) => (
            `<img class=${styles.images} src=${image} loading="lazy" alt='property'>`
            )).join('')}
          </div>
          <button class=${styles.next}>&#9654;</button>
        </div>
        <div class=${styles.row}>
          <svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' width='36' height='36' style='min-width: 36px;'><path d='M18.364 17.364L12 23.7279L5.63604 17.364C2.12132 13.8492 2.12132 8.15076 5.63604 4.63604C9.15076 1.12132 14.8492 1.12132 18.364 4.63604C21.8787 8.15076 21.8787 13.8492 18.364 17.364ZM12 13C13.1046 13 14 12.1046 14 11C14 9.89543 13.1046 9 12 9C10.8954 9 10 9.89543 10 11C10 12.1046 10.8954 13 12 13Z'></path></svg>
          <div class=${styles.neighbourhood}>${property.neighbourhood}</div>
        </div>
        <div class=${styles.details}>
          <div class=${styles.features}>
            <div>
              <i aria-hidden='true' class='fa fa-bed fa-lg bed' title='bedroom'></i>
              <span class='fa-sr-only'>bedroom</span>
              <span>${property.rooms}</span>
            </div>
            <div>
              <i aria-hidden='true' class='fa fa-bath fa-lg bath' title='bathroom'></i>
              <span class='fa-sr-only'>bathroom</span>
              <span>${property.bathrooms}</span>
            </div>
            <div>
              <i aria-hidden='true' class='fa fa-ruler fa-lg size' title='size'></i>
              <span class='fa-sr-only'>size</span>
              <span>${property.m2} m<sup>2</sup></span>
            </div>
          </div>
          <hr></hr>
          <div class=${styles.box}>
            <div class=${styles.price}>$${property.price}</div>
            <small class=${styles.address}>${property.address}</small>
          </div>
        </div>`

      const frame = content.querySelector(`.${styles.frame}`) as HTMLDivElement
      const prevButton = content.querySelector(`.${styles.prev}`) as HTMLButtonElement
      const nextButton = content.querySelector(`.${styles.next}`) as HTMLButtonElement
      const redirectButton = content.querySelector(`.${styles.link}`) as HTMLButtonElement
      const close = content.querySelector(`.${styles.corner}`) as HTMLDivElement

      function closeContainer() {
        closeActive = true
        markerView.classList.add(styles.icon)
        markerView.classList.remove(styles.property)
        markerView.content = icon
        markerView.zIndex = 0
        setTimeout(() => (closeActive = false), 0)
      }

      function updateFramePosition() {
        frame!.style.left = `-${position * imageWidth}px`
      }

      function previousImage() {
        if (position > 0) {
          position--
          updateFramePosition()
        }
      }

      function nextImage() {
        if (position < property.images.length - 1) {
          position++
          updateFramePosition()
        }
      }

      close.addEventListener('click', closeContainer)
      redirectButton!.addEventListener('click', redirect)
      prevButton!.addEventListener('click', previousImage)
      nextButton!.addEventListener('click', nextImage)

      return content
    }

    function toggleHighlight(markerView: Marker, property: Property, icon: SVGElement) {
      const content = markerView.content as HTMLElement
      const newContent = buildContent(property, markerView, icon)
      if (content.classList.contains(styles.icon)) {
        markerView.classList.remove(styles.icon)
        markerView.classList.add(styles.property) 
        markerView.content = newContent
      } 

      markerView.zIndex = 1
    }
  }
</script>

<main>
  <div id='map'></div>
</main>

<style>
  #map {
    margin: 0;
    padding: 0;
    border: none;
    width: 100vw;
    height: 100vh;
    z-index: 1;
  }
</style>

