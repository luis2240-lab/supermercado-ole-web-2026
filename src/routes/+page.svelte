
<script>
  let busqueda = '';
  let categoria = 'Todas';
  let carrito = [];
  let mensaje = '';
  let enviando = false;

  const SUPABASE_URL = 'https://xlxodjaggdwyswrloium.supabase.co';
  const SUPABASE_KEY = 'sb_publishable__PYnr5wMirAgLAx8MnvphA_7cgtqdv6';

  const categorias = [
    'Todas',
    'Frutas y verduras',
    'Carnes y pescados',
    'Lácteos',
    'Bebidas',
    'Limpieza',
    'Despensa'
  ];

  const productos = [
    { id: 1, nombre: 'Arroz selecto', categoria: 'Despensa', precio: 45, unidad: 'libra', imagen: 'https://images.unsplash.com/photo-1586201375761-83865001e31c?auto=format&fit=crop&w=700&q=80', oferta: false },
    { id: 2, nombre: 'Leche entera', categoria: 'Lácteos', precio: 85, unidad: 'unidad', imagen: 'https://images.unsplash.com/photo-1563636619-e9143da7973b?auto=format&fit=crop&w=700&q=80', oferta: false },
    { id: 3, nombre: 'Manzanas frescas', categoria: 'Frutas y verduras', precio: 120, unidad: 'libra', imagen: 'https://images.unsplash.com/photo-1560806887-1e4cd0b6cbd6?auto=format&fit=crop&w=700&q=80', oferta: true },
    { id: 4, nombre: 'Guineos maduros', categoria: 'Frutas y verduras', precio: 35, unidad: 'unidad', imagen: 'https://images.unsplash.com/photo-1571771894821-ce9b6c11b08e?auto=format&fit=crop&w=700&q=80', oferta: false },
    { id: 5, nombre: 'Pollo fresco', categoria: 'Carnes y pescados', precio: 95, unidad: 'libra', imagen: 'https://images.unsplash.com/photo-1604503468506-a8da13d82791?auto=format&fit=crop&w=700&q=80', oferta: true },
    { id: 6, nombre: 'Jugo de naranja', categoria: 'Bebidas', precio: 75, unidad: 'unidad', imagen: 'https://images.unsplash.com/photo-1600271886742-f049cd451bba?auto=format&fit=crop&w=700&q=80', oferta: false },
    { id: 7, nombre: 'Queso blanco', categoria: 'Lácteos', precio: 110, unidad: 'paquete', imagen: 'https://images.unsplash.com/photo-1486297678162-eb2a19b0a32d?auto=format&fit=crop&w=700&q=80', oferta: false },
    { id: 8, nombre: 'Aceite vegetal', categoria: 'Despensa', precio: 135, unidad: 'botella', imagen: 'https://images.unsplash.com/photo-1474979266404-7eaacbcd87c5?auto=format&fit=crop&w=700&q=80', oferta: false },
    { id: 9, nombre: 'Detergente', categoria: 'Limpieza', precio: 150, unidad: 'unidad', imagen: 'https://images.unsplash.com/photo-1585421514738-01798e348b17?auto=format&fit=crop&w=700&q=80', oferta: true },
    { id: 10, nombre: 'Tomates frescos', categoria: 'Frutas y verduras', precio: 55, unidad: 'libra', imagen: 'https://images.unsplash.com/photo-1546094096-0df4bcaaa337?auto=format&fit=crop&w=700&q=80', oferta: false },
    { id: 11, nombre: 'Agua mineral', categoria: 'Bebidas', precio: 40, unidad: 'botella', imagen: 'https://images.unsplash.com/photo-1523362628745-0c100150b504?auto=format&fit=crop&w=700&q=80', oferta: false },
    { id: 12, nombre: 'Habichuelas', categoria: 'Despensa', precio: 65, unidad: 'paquete', imagen: 'https://images.unsplash.com/photo-1515543904379-3d757afe72e4?auto=format&fit=crop&w=700&q=80', oferta: false }
  ];

  
let productosFiltrados = $derived(
  productos.filter((producto) => {
    const coincideBusqueda = producto.nombre.toLowerCase().includes(busqueda.toLowerCase());
    const coincideCategoria = categoria === 'Todas' || producto.categoria === categoria;
    return coincideBusqueda && coincideCategoria;
  })
);


 
let total = $derived(
  carrito.reduce((suma, producto) => suma + producto.precio * producto.cantidad, 0)
);

let cantidadCarrito = $derived(
  carrito.reduce((suma, producto) => suma + producto.cantidad, 0)
);

let ofertas = $derived(
  productos.filter((producto) => producto.oferta)
);


  function agregarAlCarrito(producto) {
    const existente = carrito.find((item) => item.id === producto.id);

    if (existente) {
      carrito = carrito.map((item) =>
        item.id === producto.id
          ? { ...item, cantidad: item.cantidad + 1 }
          : item
      );
    } else {
      carrito = [...carrito, { ...producto, cantidad: 1 }];
    }
  }

  function cambiarCantidad(id, cambio) {
    carrito = carrito
      .map((item) =>
        item.id === id
          ? { ...item, cantidad: item.cantidad + cambio }
          : item
      )
      .filter((item) => item.cantidad > 0);
  }

  function quitarDelCarrito(id) {
    carrito = carrito.filter((item) => item.id !== id);
  }

  function dinero(valor) {
    return new Intl.NumberFormat('es-DO', {
      style: 'currency',
      currency: 'DOP',
      maximumFractionDigits: 2
    }).format(valor);
  }

  async function enviar(event) {
    event.preventDefault();

    const formulario = event.currentTarget;
    const datos = Object.fromEntries(new FormData(formulario));

    mensaje = '';
    enviando = true;

    try {
      const respuesta = await fetch(`${SUPABASE_URL}/rest/v1/contactos`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'apikey': SUPABASE_KEY,
          'Prefer': 'return=minimal'
        },
        body: JSON.stringify({
          nombre: String(datos.nombre).trim(),
          email: String(datos.email).trim(),
          motivo: String(datos.motivo).trim(),
          mensaje: String(datos.mensaje).trim()
        })
      });

      if (!respuesta.ok) {
        const error = await respuesta.json().catch(() => ({}));
        console.error('Error de Supabase:', error);
        throw new Error(error.message || 'No se pudo enviar el mensaje. Revisa la configuración de Supabase.');
      }

      mensaje = '¡Mensaje enviado correctamente!';
      formulario.reset();
    } catch (error) {
      mensaje = error.message || 'Ocurrió un error al enviar el mensaje.';
    } finally {
      enviando = false;
    }
  }
</script>

<svelte:head>
  <title>Supermercado Olé | Todo para tu hogar</title>
  <meta name="description" content="Encuentra productos, ofertas y artículos para tu hogar en Supermercado Olé, República Dominicana." />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
</svelte:head>

<div class="pagina">
  <div class="barra-superior">
    <span>📍 República Dominicana</span>
    <span>Horario general: todos los días, 7:00 a. m. – 10:00 p. m.</span>
  </div>

  <header class="encabezado">
    <a class="logo" href="#inicio" aria-label="Supermercado Olé, inicio">
      <span class="logo-icono">olé<span>.</span></span>
      <span class="logo-texto">SUPERMERCADO</span>
    </a>

    <nav class="navegacion">
      <a href="#inicio">Inicio</a>
      <a href="#catalogo">Productos</a>
      <a href="#ofertas">Ofertas</a>
      <a href="#sucursales">Sucursales</a>
      <a href="#contacto">Contáctanos</a>
    </nav>

    <div class="acciones-header">
      <a class="login" href="#contacto">Iniciar sesión</a>
      <a class="carrito-header" href="#carrito">
        🛒 <span>Mi carrito</span>
        <b>{cantidadCarrito}</b>
      </a>
    </div>
  </header>

  <main>
    <section class="hero" id="inicio">
      <div class="hero-contenido">
        <span class="etiqueta">TODO PARA TU HOGAR</span>
        <h1>Tu compra diaria,<br /><span>más fácil y cerca de ti.</span></h1>
        <p>Encuentra tus productos favoritos, descubre ofertas y organiza tu próxima compra con Supermercado Olé.</p>

        <form class="busqueda-hero" on:submit|preventDefault={() => document.getElementById('catalogo')?.scrollIntoView({ behavior: 'smooth' })}>
          <span>⌕</span>
          <input
            bind:value={busqueda}
            type="search"
            placeholder="¿Qué necesitas hoy?"
            aria-label="Buscar productos"
          />
          <button type="submit">Buscar</button>
        </form>

        <div class="hero-beneficios">
          <span>✓ Productos para tu hogar</span>
          <span>✓ Ofertas especiales</span>
        </div>
      </div>

      <div class="hero-imagen">
        <img src="https://images.unsplash.com/photo-1542838132-92c53300491e?auto=format&fit=crop&w=1100&q=85" alt="Frutas y verduras frescas en un supermercado" />
        <div class="tarjeta-flotante">
          <span class="tarjeta-icono">🛍️</span>
          <div>
            <strong>Tu compra comienza aquí</strong>
            <p>Todo lo que necesitas en un solo lugar.</p>
          </div>
        </div>
      </div>
    </section>

    <section class="categorias-seccion">
      <div class="titulo-seccion">
        <div>
          <span class="etiqueta">COMPRA A TU MANERA</span>
          <h2>Explora las categorías</h2>
        </div>
        <a href="#catalogo">Ver productos →</a>
      </div>

      <div class="categorias-grid">
        {#each categorias.filter((item) => item !== 'Todas') as item}
          <button
            class:categoria-activa={categoria === item}
            class="categoria-card"
            on:click={() => { categoria = item; document.getElementById('catalogo')?.scrollIntoView({ behavior: 'smooth' }); }}
          >
            <span class="categoria-icono">
              {item === 'Frutas y verduras' ? '🥑' : item === 'Carnes y pescados' ? '🍗' : item === 'Lácteos' ? '🥛' : item === 'Bebidas' ? '🧃' : item === 'Limpieza' ? '🧼' : '🛒'}
            </span>
            <strong>{item}</strong>
            <span class="flecha">↗</span>
          </button>
        {/each}
      </div>
    </section>

    <section class="productos-seccion" id="catalogo">
      <div class="titulo-seccion">
        <div>
          <span class="etiqueta">NUESTRO CATÁLOGO</span>
          <h2>Productos para ti</h2>
          <p class="subtitulo">Busca y agrega tus productos favoritos al carrito.</p>
        </div>
        <div class="filtro-categoria">
          <label for="filtro">Categoría</label>
          <select id="filtro" bind:value={categoria}>
            {#each categorias as item}
              <option value={item}>{item}</option>
            {/each}
          </select>
        </div>
      </div>

      <div class="busqueda-catalogo">
        <span>⌕</span>
        <input bind:value={busqueda} type="search" placeholder="Buscar por nombre del producto..." aria-label="Buscar en el catálogo" />
        {#if busqueda}
          <button class="limpiar" on:click={() => busqueda = ''}>Limpiar</button>
        {/if}
      </div>

      {#if productosFiltrados.length > 0}
        <div class="productos-grid">
          {#each productosFiltrados as producto}
            <article class="producto-card">
              <div class="producto-imagen">
                <img src={producto.imagen} alt={producto.nombre} loading="lazy" />
                {#if producto.oferta}
                  <span class="etiqueta-oferta">OFERTA</span>
                {/if}
              </div>
              <div class="producto-info">
                <span class="producto-categoria">{producto.categoria}</span>
                <h3>{producto.nombre}</h3>
                <p class="unidad">Precio por {producto.unidad}</p>
                <div class="producto-pie">
                  <strong>{dinero(producto.precio)}</strong>
                  <button class="agregar" on:click={() => agregarAlCarrito(producto)} aria-label={`Agregar ${producto.nombre} al carrito`}>+</button>
                </div>
              </div>
            </article>
          {/each}
        </div>
      {:else}
        <div class="sin-resultados">
          <span>🔎</span>
          <h3>No encontramos productos</h3>
          <p>Prueba con otro nombre o selecciona otra categoría.</p>
          <button class="boton-secundario" on:click={() => { busqueda = ''; categoria = 'Todas'; }}>Mostrar todos</button>
        </div>
      {/if}
    </section>

    <section class="ofertas-seccion" id="ofertas">
      <div class="ofertas-cabecera">
        <div>
          <span class="etiqueta etiqueta-blanca">AHORRA EN TU COMPRA</span>
          <h2>Ofertas que te encantarán</h2>
          <p>Descubre algunos de los productos destacados de nuestra selección.</p>
        </div>
        <span class="oferta-grafica">%</span>
      </div>

      <div class="ofertas-grid">
        {#each ofertas as producto}
          <article class="oferta-card">
            <img src={producto.imagen} alt={producto.nombre} loading="lazy" />
            <div class="oferta-info">
              <span>OFERTA DESTACADA</span>
              <h3>{producto.nombre}</h3>
              <strong>{dinero(producto.precio)}</strong>
              <button on:click={() => agregarAlCarrito(producto)}>Agregar al carrito +</button>
            </div>
          </article>
        {/each}
      </div>
    </section>

    <section class="carrito-seccion" id="carrito">
      <div class="titulo-seccion">
        <div>
          <span class="etiqueta">ORGANIZA TU COMPRA</span>
          <h2>Mi carrito de compras</h2>
        </div>
        <span class="contador-carrito">{cantidadCarrito} producto(s)</span>
      </div>

      {#if carrito.length === 0}
        <div class="carrito-vacio">
          <span>🛒</span>
          <h3>Tu carrito está vacío</h3>
          <p>Agrega productos del catálogo para comenzar tu compra.</p>
          <a href="#catalogo" class="boton-rojo">Ver productos</a>
        </div>
      {:else}
        <div class="carrito-contenido">
          <div class="lista-carrito">
            {#each carrito as producto}
              <div class="item-carrito">
                <img src={producto.imagen} alt={producto.nombre} />
                <div class="item-detalle">
                  <strong>{producto.nombre}</strong>
                  <span>{dinero(producto.precio)} por {producto.unidad}</span>
                  <button class="quitar" on:click={() => quitarDelCarrito(producto.id)}>Eliminar</button>
                </div>
                <div class="cantidad-control">
                  <button on:click={() => cambiarCantidad(producto.id, -1)} aria-label="Reducir cantidad">−</button>
                  <span>{producto.cantidad}</span>
                  <button on:click={() => cambiarCantidad(producto.id, 1)} aria-label="Aumentar cantidad">+</button>
                </div>
                <strong class="subtotal">{dinero(producto.precio * producto.cantidad)}</strong>
              </div>
            {/each}
          </div>

          <aside class="resumen-carrito">
            <h3>Resumen de compra</h3>
            <div class="resumen-linea"><span>Productos</span><span>{cantidadCarrito}</span></div>
            <div class="resumen-linea"><span>Subtotal</span><strong>{dinero(total)}</strong></div>
            <div class="resumen-total"><span>Total estimado</span><strong>{dinero(total)}</strong></div>
            <p>Este catálogo es una demostración. Los precios y productos pueden variar según la sucursal.</p>
            <a href="#contacto" class="boton-rojo boton-ancho">Consultar mi compra</a>
          </aside>
        </div>
      {/if}
    </section>

    <section class="sucursales-seccion" id="sucursales">
      <div class="sucursal-imagen">
        <img src="https://images.unsplash.com/photo-1604719312566-8912e9c8a213?auto=format&fit=crop&w=1000&q=85" alt="Pasillos de un supermercado" loading="lazy" />
      </div>
      <div class="sucursal-texto">
        <span class="etiqueta">CERCA DE TI</span>
        <h2>Encuentra tu Olé más cercano</h2>
        <p>Visita nuestras sucursales en República Dominicana y encuentra los productos que necesitas para tu hogar.</p>
        <div class="sucursal-dato"><span>📍</span><div><strong>Varias ubicaciones</strong><p>Consulta la dirección de tu sucursal preferida.</p></div></div>
        <div class="sucursal-dato"><span>🕒</span><div><strong>Horario general</strong><p>Todos los días, de 7:00 a. m. a 10:00 p. m. Puede variar por sucursal.</p></div></div>
        <a href="#contacto" class="boton-rojo">Solicitar información</a>
      </div>
    </section>

    <section class="contacto" id="contacto">
      <div class="contacto-interior">
        <div class="contacto-texto">
          <span class="etiqueta etiqueta-blanca">ESTAMOS PARA AYUDARTE</span>
          <h2>¿Tienes alguna pregunta?</h2>
          <p>Queremos conocer tus dudas y ayudarte a encontrar la información que necesitas sobre nuestros productos, ofertas y sucursales.</p>

          <div class="contacto-dato"><span>📍</span><div><strong>República Dominicana</strong><p>Supermercado Olé</p></div></div>
          <div class="contacto-dato"><span>💬</span><div><strong>Atención a nuestros clientes</strong><p>Escríbenos mediante este formulario.</p></div></div>
        </div>

        <form class="formulario" on:submit={enviar}>
          <h3>Contáctanos</h3>
          <p class="formulario-descripcion">Completa los campos y envíanos tu mensaje.</p>

          <label for="nombre">Nombre completo
            <input id="nombre" name="nombre" type="text" required minlength="2" maxlength="100" placeholder="Escribe tu nombre" autocomplete="name" />
          </label>

          <label for="email">Correo electrónico
            <input id="email" name="email" type="email" required maxlength="254" placeholder="nombre@correo.com" autocomplete="email" />
          </label>

          <label for="motivo">Motivo
            <select id="motivo" name="motivo" required>
              <option value="">Selecciona un motivo</option>
              <option value="Productos">Productos</option>
              <option value="Ofertas y promociones">Ofertas y promociones</option>
              <option value="Sucursales y horarios">Sucursales y horarios</option>
              <option value="Pedidos">Pedidos</option>
              <option value="Otro">Otro</option>
            </select>
          </label>

          <label for="mensaje">Mensaje
            <textarea id="mensaje" name="mensaje" required minlength="10" maxlength="3000" rows="4" placeholder="¿Cómo podemos ayudarte?"></textarea>
          </label>

          <button class="boton-rojo boton-ancho" type="submit" disabled={enviando}>
            {enviando ? 'Enviando...' : 'Enviar mensaje →'}
          </button>

          <p class="estado" class:exito={mensaje.startsWith('¡')} class:error={mensaje !== '' && !mensaje.startsWith('¡')} role="status">{mensaje}</p>
        </form>
      </div>
    </section>
  </main>

  <footer class="pie-pagina">
    <div class="pie-principal">
      <a class="logo logo-pie" href="#inicio">
        <span class="logo-icono">olé<span>.</span></span>
        <span class="logo-texto">SUPERMERCADO</span>
      </a>
      <p>Todo para tu hogar, más fácil y cerca de ti.</p>
      <div class="pie-enlaces">
        <a href="#inicio">Inicio</a>
        <a href="#catalogo">Productos</a>
        <a href="#ofertas">Ofertas</a>
        <a href="#contacto">Contáctanos</a>
      </div>
    </div>
    <div class="pie-final">
      <span>© 2026 Supermercado Olé. Proyecto educativo.</span>
      <a href="#inicio">Volver arriba ↑</a>
    </div>
  </footer>
</div>

<style>
  :global(*) {
    box-sizing: border-box;
  }

  :global(html) {
    scroll-behavior: smooth;
    scroll-padding-top: 85px;
  }

  :global(body) {
    margin: 0;
    background: #ffffff;
    color: #252525;
    font-family: Arial, Helvetica, sans-serif;
  }

  :global(button),
  :global(input),
  :global(select),
  :global(textarea) {
    font: inherit;
  }

  :global(button),
  :global(a) {
    -webkit-tap-highlight-color: transparent;
  }

  :global(a) {
    color: inherit;
    text-decoration: none;
  }

  :global(button) {
    cursor: pointer;
  }

  .pagina {
    width: 100%;
    overflow: hidden;
  }

  .barra-superior {
    min-height: 34px;
    padding: 8px 5%;
    display: flex;
    justify-content: space-between;
    gap: 12px;
    background: #252525;
    color: #ffffff;
    font-size: 11px;
  }

  .encabezado {
    min-height: 82px;
    padding: 15px 5%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 22px;
    background: #ffffff;
    border-bottom: 1px solid #eeeeee;
  }

  .logo {
    display: inline-flex;
    flex-direction: column;
    flex-shrink: 0;
    line-height: 1;
  }

  .logo-icono {
    color: #d7192d;
    font-size: 37px;
    font-weight: 900;
    letter-spacing: -3px;
  }

  .logo-icono span {
    color: #efb323;
  }

  .logo-texto {
    margin-top: 4px;
    font-size: 8px;
    font-weight: 800;
    letter-spacing: 2px;
  }

  .navegacion {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    gap: 22px;
    font-size: 12px;
    font-weight: 700;
  }

  .navegacion a:hover,
  .pie-enlaces a:hover {
    color: #d7192d;
  }

  .acciones-header {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-shrink: 0;
  }

  .login {
    border: 1px solid #dddddd;
    border-radius: 7px;
    padding: 11px 13px;
    font-size: 11px;
    font-weight: 700;
  }

  .login:hover {
    border-color: #d7192d;
    color: #d7192d;
  }

  .carrito-header {
    display: flex;
    align-items: center;
    gap: 7px;
    font-size: 12px;
    font-weight: 700;
  }

  .carrito-header b {
    display: grid;
    place-items: center;
    width: 23px;
    height: 23px;
    border-radius: 50%;
    background: #d7192d;
    color: #ffffff;
    font-size: 11px;
  }

  .hero {
    min-height: 460px;
    padding: 55px 7%;
    display: grid;
    grid-template-columns: 1.05fr 0.95fr;
    align-items: center;
    gap: 50px;
    background: #fff8f1;
  }

  .hero-contenido {
    max-width: 590px;
  }

  .etiqueta {
    display: inline-block;
    color: #d7192d;
    font-size: 10px;
    font-weight: 900;
    letter-spacing: 1.8px;
  }

  .hero h1 {
    margin: 18px 0;
    font-size: clamp(34px, 4.4vw, 57px);
    line-height: 1.08;
    letter-spacing: -2px;
  }

  .hero h1 span {
    color: #d7192d;
  }

  .hero-contenido > p {
    max-width: 460px;
    color: #666666;
    font-size: 14px;
    line-height: 1.8;
  }

  .busqueda-hero {
    margin-top: 25px;
    max-width: 510px;
    padding: 7px;
    display: flex;
    align-items: center;
    gap: 10px;
    background: #ffffff;
    border: 1px solid #eeeeee;
    border-radius: 10px;
    box-shadow: 0 8px 24px #0000000b;
  }

  .busqueda-hero > span,
  .busqueda-catalogo > span {
    padding-left: 10px;
    color: #777777;
    font-size: 24px;
  }

  .busqueda-hero input,
  .busqueda-catalogo input {
    width: 100%;
    min-width: 0;
    padding: 12px 4px;
    border: 0;
    outline: none;
    background: #ffffff;
    color: #222222;
    font-size: 12px;
  }

  .busqueda-hero input::placeholder,
  .busqueda-catalogo input::placeholder {
    color: #777777;
    opacity: 1;
  }

  .busqueda-hero button {
    flex-shrink: 0;
    padding: 13px 21px;
    border: 0;
    border-radius: 7px;
    background: #d7192d;
    color: #ffffff;
    font-size: 12px;
    font-weight: 800;
  }

  .hero-beneficios {
    margin-top: 20px;
    display: flex;
    flex-wrap: wrap;
    gap: 17px;
    color: #555555;
    font-size: 11px;
  }

  .hero-beneficios span:first-child::first-letter {
    color: #16834a;
  }

  .hero-imagen {
    position: relative;
    min-height: 330px;
  }

  .hero-imagen > img {
    width: 100%;
    height: 360px;
    display: block;
    object-fit: cover;
    border-radius: 20px;
  }

  .tarjeta-flotante {
    position: absolute;
    right: -16px;
    bottom: 20px;
    max-width: 285px;
    padding: 17px;
    display: flex;
    align-items: center;
    gap: 12px;
    border-radius: 12px;
    background: #ffffff;
    box-shadow: 0 12px 35px #00000019;
  }

  .tarjeta-icono {
    font-size: 28px;
  }

  .tarjeta-flotante strong {
    font-size: 12px;
  }

  .tarjeta-flotante p {
    margin: 5px 0 0;
    color: #777777;
    font-size: 10px;
    line-height: 1.5;
  }

  .categorias-seccion,
  .productos-seccion,
  .carrito-seccion {
    padding: 65px 7%;
  }

  .titulo-seccion {
    margin-bottom: 27px;
    display: flex;
    align-items: end;
    justify-content: space-between;
    gap: 20px;
  }

  .titulo-seccion h2,
  .ofertas-cabecera h2,
  .sucursal-texto h2,
  .contacto-texto h2 {
    margin: 10px 0;
    font-size: clamp(25px, 3vw, 35px);
    letter-spacing: -1px;
  }

  .titulo-seccion > a {
    color: #d7192d;
    font-size: 12px;
    font-weight: 800;
  }

  .categorias-grid {
    display: grid;
    grid-template-columns: repeat(6, minmax(0, 1fr));
    gap: 14px;
  }

  .categoria-card {
    position: relative;
    min-height: 145px;
    padding: 20px 12px;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
    gap: 15px;
    border: 1px solid #eeeeee;
    border-radius: 12px;
    background: #ffffff;
    color: #252525;
    text-align: left;
    transition: transform 0.2s, border-color 0.2s;
  }

  .categoria-card:hover,
  .categoria-activa {
    transform: translateY(-3px);
    border-color: #d7192d;
  }

  .categoria-icono {
    font-size: 30px;
  }

  .categoria-card strong {
    font-size: 11px;
    line-height: 1.4;
  }

  .flecha {
    position: absolute;
    top: 13px;
    right: 13px;
    color: #999999;
  }

  .subtitulo {
    margin: 7px 0 0;
    color: #777777;
    font-size: 12px;
  }

  .productos-seccion {
    background: #fafafa;
  }

  .filtro-categoria {
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 11px;
    font-weight: 700;
  }

  .filtro-categoria select {
    max-width: 220px;
    padding: 11px;
    border: 1px solid #dddddd;
    border-radius: 7px;
    background: #ffffff;
    color: #222222;
  }

  .busqueda-catalogo {
    margin-bottom: 25px;
    padding: 4px;
    display: flex;
    align-items: center;
    gap: 10px;
    border: 1px solid #e5e5e5;
    border-radius: 8px;
    background: #ffffff;
  }

  .limpiar {
    margin-right: 8px;
    border: 0;
    background: transparent;
    color: #d7192d;
    font-size: 11px;
    font-weight: 700;
  }

  .productos-grid {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 20px;
  }

  .producto-card {
    overflow: hidden;
    border: 1px solid #eeeeee;
    border-radius: 12px;
    background: #ffffff;
    transition: transform 0.2s, box-shadow 0.2s;
  }

  .producto-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 12px 28px #0000000c;
  }

  .producto-imagen {
    position: relative;
    height: 180px;
    overflow: hidden;
    background: #f4f4f4;
  }

  .producto-imagen img,
  .oferta-card > img {
    width: 100%;
    height: 100%;
    display: block;
    object-fit: cover;
  }

  .etiqueta-oferta {
    position: absolute;
    top: 12px;
    left: 12px;
    padding: 7px 10px;
    border-radius: 5px;
    background: #d7192d;
    color: #ffffff;
    font-size: 9px;
    font-weight: 900;
  }

  .producto-info {
    padding: 16px;
  }

  .producto-categoria {
    color: #777777;
    font-size: 10px;
  }

  .producto-info h3 {
    margin: 8px 0;
    font-size: 15px;
  }

  .unidad {
    margin: 0;
    color: #777777;
    font-size: 10px;
  }

  .producto-pie {
    margin-top: 16px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
  }

  .producto-pie strong {
    color: #d7192d;
    font-size: 17px;
  }

  .agregar {
    width: 35px;
    height: 35px;
    border: 0;
    border-radius: 8px;
    background: #d7192d;
    color: #ffffff;
    font-size: 24px;
  }

  .agregar:hover,
  .boton-rojo:hover,
  .busqueda-hero button:hover {
    background: #ad1121;
  }

  .sin-resultados,
  .carrito-vacio {
    padding: 45px 20px;
    border: 1px dashed #dddddd;
    border-radius: 12px;
    text-align: center;
    background: #ffffff;
  }

  .sin-resultados > span,
  .carrito-vacio > span {
    font-size: 35px;
  }

  .sin-resultados h3,
  .carrito-vacio h3 {
    margin-bottom: 8px;
  }

  .sin-resultados p,
  .carrito-vacio p {
    color: #777777;
    font-size: 12px;
  }

  .boton-secundario {
    margin-top: 10px;
    padding: 12px 17px;
    border: 1px solid #d7192d;
    border-radius: 7px;
    background: #ffffff;
    color: #d7192d;
    font-size: 12px;
    font-weight: 800;
  }

  .ofertas-seccion {
    padding: 55px 7%;
    background: #d7192d;
    color: #ffffff;
  }

  .etiqueta-blanca {
    color: #ffffff;
  }

  .ofertas-cabecera {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 20px;
  }

  .ofertas-cabecera h2 {
    margin-top: 12px;
  }

  .ofertas-cabecera p {
    color: #ffffffd9;
    font-size: 12px;
  }

  .oferta-grafica {
    font-size: 90px;
    font-weight: 900;
    opacity: 0.2;
  }

  .ofertas-grid {
    margin-top: 25px;
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 17px;
  }

  .oferta-card {
    overflow: hidden;
    display: grid;
    grid-template-columns: 115px 1fr;
    border-radius: 10px;
    background: #ffffff;
    color: #252525;
  }

  .oferta-card > img {
    min-height: 180px;
  }

  .oferta-info {
    padding: 15px;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
  }

  .oferta-info > span {
    color: #d7192d;
    font-size: 8px;
    font-weight: 900;
    letter-spacing: 1px;
  }

  .oferta-info h3 {
    margin: 9px 0;
    font-size: 14px;
  }

  .oferta-info strong {
    color: #d7192d;
    font-size: 17px;
  }

  .oferta-info button {
    margin-top: 13px;
    padding: 9px 10px;
    border: 0;
    border-radius: 5px;
    background: #252525;
    color: #ffffff;
    font-size: 10px;
    font-weight: 700;
  }

  .contador-carrito {
    padding: 9px 13px;
    border-radius: 20px;
    background: #f6eeee;
    color: #d7192d;
    font-size: 11px;
    font-weight: 800;
  }

  .boton-rojo {
    display: inline-block;
    padding: 13px 18px;
    border: 0;
    border-radius: 7px;
    background: #d7192d;
    color: #ffffff;
    text-align: center;
    font-size: 12px;
    font-weight: 800;
    cursor: pointer;
  }

  .boton-ancho {
    width: 100%;
  }

  .carrito-contenido {
    display: grid;
    grid-template-columns: 1.6fr 0.8fr;
    align-items: start;
    gap: 25px;
  }

  .lista-carrito {
    border: 1px solid #eeeeee;
    border-radius: 10px;
  }

  .item-carrito {
    padding: 15px;
    display: grid;
    grid-template-columns: 65px minmax(0, 1fr) auto auto;
    align-items: center;
    gap: 15px;
    border-bottom: 1px solid #eeeeee;
  }

  .item-carrito:last-child {
    border-bottom: 0;
  }

  .item-carrito > img {
    width: 65px;
    height: 65px;
    border-radius: 7px;
    object-fit: cover;
  }

  .item-detalle {
    display: flex;
    flex-direction: column;
    gap: 6px;
  }

  .item-detalle strong,
  .subtotal {
    font-size: 12px;
  }

  .item-detalle > span {
    color: #777777;
    font-size: 10px;
  }

  .quitar {
    width: fit-content;
    padding: 0;
    border: 0;
    background: transparent;
    color: #d7192d;
    font-size: 10px;
  }

  .cantidad-control {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .cantidad-control button {
    width: 27px;
    height: 27px;
    border: 1px solid #dddddd;
    border-radius: 5px;
    background: #ffffff;
    color: #222222;
  }

  .cantidad-control span {
    font-size: 12px;
  }

  .resumen-carrito {
    padding: 22px;
    border: 1px solid #eeeeee;
    border-radius: 10px;
  }

  .resumen-carrito h3 {
    margin-top: 0;
    font-size: 17px;
  }

  .resumen-linea,
  .resumen-total {
    padding: 13px 0;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 10px;
    border-bottom: 1px solid #eeeeee;
    font-size: 12px;
  }

  .resumen-total {
    margin-bottom: 13px;
    border-bottom: 0;
    font-size: 14px;
    font-weight: 800;
  }

  .resumen-total strong {
    color: #d7192d;
    font-size: 20px;
  }

  .resumen-carrito > p {
    color: #777777;
    font-size: 10px;
    line-height: 1.7;
  }

  .sucursales-seccion {
    padding: 65px 7%;
    display: grid;
    grid-template-columns: 1fr 1fr;
    align-items: center;
    gap: 55px;
    background: #f8f8f8;
  }

  .sucursal-imagen img {
    width: 100%;
    height: 370px;
    display: block;
    object-fit: cover;
    border-radius: 14px;
  }

  .sucursal-texto > p {
    color: #666666;
    font-size: 13px;
    line-height: 1.8;
  }

  .sucursal-dato {
    margin: 20px 0;
    display: flex;
    align-items: flex-start;
    gap: 13px;
  }

  .sucursal-dato > span {
    font-size: 22px;
  }

  .sucursal-dato strong {
    font-size: 12px;
  }

  .sucursal-dato p {
    margin: 5px 0 0;
    color: #777777;
    font-size: 11px;
    line-height: 1.6;
  }

  .contacto {
    padding: 65px 7%;
    background: #252525;
    color: #ffffff;
  }

  .contacto-interior {
    display: grid;
    grid-template-columns: 1fr 0.85fr;
    align-items: start;
    gap: 55px;
  }

  .contacto-texto {
    padding-top: 25px;
  }

  .contacto-texto h2 {
    max-width: 420px;
    margin-top: 18px;
    font-size: clamp(30px, 3.5vw, 43px);
    line-height: 1.15;
  }

  .contacto-texto > p {
    max-width: 460px;
    color: #f0f0f0;
    font-size: 14px;
    line-height: 1.9;
  }

  .contacto-dato {
    margin-top: 28px;
    display: flex;
    align-items: flex-start;
    gap: 13px;
  }

  .contacto-dato > span {
    font-size: 22px;
  }

  .contacto-dato strong {
    color: #ffffff;
    font-size: 13px;
  }

  .contacto-dato p {
    margin: 6px 0 0;
    color: #f0f0f0;
    font-size: 12px;
  }

  .formulario {
    min-width: 0;
    padding: 28px;
    display: grid;
    gap: 16px;
    border-radius: 13px;
    background: #ffffff;
    color: #222222;
    box-shadow: 0 15px 45px #00000020;
  }

  .formulario h3 {
    margin: 0;
    color: #222222;
    font-size: 23px;
  }

  .formulario-descripcion {
    margin: -8px 0 0;
    color: #555555;
    font-size: 12px;
    line-height: 1.6;
  }

  .formulario label {
    display: grid;
    gap: 8px;
    color: #222222;
    font-size: 12px;
    font-weight: 700;
  }

  .formulario input,
  .formulario select,
  .formulario textarea {
    width: 100%;
    min-width: 0;
    padding: 13px;
    border: 1px solid #bdbdbd;
    border-radius: 8px;
    outline: none;
    background-color: #ffffff !important;
    color: #222222 !important;
    -webkit-text-fill-color: #222222;
    caret-color: #d7192d;
    font-size: 14px;
    font-weight: 400;
    line-height: 1.5;
    opacity: 1;
  }

  .formulario input::placeholder,
  .formulario textarea::placeholder {
    color: #777777 !important;
    -webkit-text-fill-color: #777777;
    opacity: 1;
  }

  .formulario input:focus,
  .formulario select:focus,
  .formulario textarea:focus {
    border-color: #d7192d;
    outline: 2px solid #d7192d;
    outline-offset: 1px;
  }

  .formulario input::selection,
  .formulario textarea::selection {
    color: #ffffff;
    background-color: #d7192d;
    -webkit-text-fill-color: #ffffff;
  }

  .formulario textarea {
    min-height: 115px;
    resize: vertical;
  }

  .formulario button:disabled {
    cursor: wait;
    opacity: 0.65;
  }

  .estado {
    min-height: 18px;
    margin: 0;
    font-size: 12px;
    line-height: 1.6;
    overflow-wrap: anywhere;
  }

  .estado.exito {
    color: #147a3c;
  }

  .estado.error {
    color: #b00020;
  }

  .pie-pagina {
    padding: 35px 7% 0;
    background: #171717;
    color: #ffffff;
  }

  .pie-principal {
    padding-bottom: 30px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 20px;
  }

  .logo-pie .logo-texto {
    color: #ffffff;
  }

  .pie-principal > p {
    color: #cccccc;
    font-size: 11px;
  }

  .pie-enlaces {
    display: flex;
    flex-wrap: wrap;
    gap: 17px;
    font-size: 11px;
  }

  .pie-final {
    padding: 18px 0;
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 12px;
    border-top: 1px solid #ffffff20;
    color: #cccccc;
    font-size: 10px;
  }

  .pie-final a {
    color: #ffffff;
  }

  @media (max-width: 1000px) {
    .encabezado {
      flex-wrap: wrap;
    }

    .navegacion {
      order: 3;
      width: 100%;
      justify-content: flex-start;
    }

    .categorias-grid {
      grid-template-columns: repeat(3, minmax(0, 1fr));
    }

    .productos-grid {
      grid-template-columns: repeat(3, minmax(0, 1fr));
    }

    .ofertas-grid {
      grid-template-columns: 1fr;
    }

    .oferta-card {
      grid-template-columns: 130px 1fr;
    }
  }

  @media (max-width: 700px) {
    .barra-superior {
      flex-direction: column;
      gap: 5px;
      font-size: 10px;
    }

    .encabezado {
      padding: 15px 5%;
      gap: 15px;
    }

    .logo-icono {
      font-size: 31px;
    }

    .acciones-header {
      gap: 8px;
    }

    .login {
      padding: 9px;
      font-size: 10px;
    }

    .carrito-header span {
      display: none;
    }

    .navegacion {
      gap: 15px;
      font-size: 11px;
    }

    .hero {
      padding: 40px 6%;
      grid-template-columns: 1fr;
      gap: 30px;
    }

    .hero h1 {
      font-size: 38px;
    }

    .hero-imagen {
      min-height: 250px;
    }

    .hero-imagen > img {
      height: 280px;
    }

    .tarjeta-flotante {
      right: 8px;
      bottom: 12px;
    }

    .categorias-seccion,
    .productos-seccion,
    .carrito-seccion,
    .ofertas-seccion,
    .sucursales-seccion,
    .contacto {
      padding: 45px 6%;
    }

    .titulo-seccion {
      align-items: flex-start;
      flex-direction: column;
      gap: 13px;
    }

    .categorias-grid {
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 10px;
    }

    .categoria-card {
      min-height: 120px;
    }

    .productos-grid {
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 12px;
    }

    .producto-imagen {
      height: 145px;
    }

    .producto-info {
      padding: 12px;
    }

    .producto-info h3 {
      font-size: 13px;
    }

    .producto-pie strong {
      font-size: 14px;
    }

    .filtro-categoria {
      width: 100%;
      justify-content: space-between;
    }

    .filtro-categoria select {
      max-width: 65%;
    }

    .oferta-grafica {
      font-size: 55px;
    }

    .carrito-contenido,
    .sucursales-seccion,
    .contacto-interior {
      grid-template-columns: 1fr;
      gap: 25px;
    }

    .item-carrito {
      grid-template-columns: 52px minmax(0, 1fr) auto;
      gap: 10px;
    }

    .item-carrito > img {
      width: 52px;
      height: 52px;
    }

    .cantidad-control {
      grid-column: 2;
      grid-row: 2;
    }

    .subtotal {
      grid-column: 3;
      grid-row: 1 / span 2;
      align-self: center;
    }

    .sucursal-imagen img {
      height: 260px;
    }

    .contacto-texto {
      padding-top: 0;
    }

    .formulario {
      padding: 21px;
    }

    .pie-principal {
      align-items: flex-start;
      flex-direction: column;
    }
  }

  @media (max-width: 380px) {
    .productos-grid {
      grid-template-columns: 1fr;
    }

    .producto-imagen {
      height: 210px;
    }

    .navegacion {
      gap: 11px;
    }

    .hero h1 {
      font-size: 32px;
    }
  }
</style>


