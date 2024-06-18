<script lang="ts">
    export let coffee: {
        blend_name: string;
        origin: string;
        variety: string;
        notes: string;
        intensifier: string;
    };
    export let getTagColor: (tag: string) => string;

    let imageLoaded = false;
    let imageUrl = `https://loremflickr.com/500/500/coffee_bean?random=${Math.random()}`;
    let placeholderUrl = 'placeholder.gif';
    let isImageLoaded = false;

    function loadImage(url: string) {
        const img = new Image();
        img.src = url;
        img.onload = () => {
            imageUrl = url;
            isImageLoaded = true;
        };
        img.onerror = () => {
            imageUrl = placeholderUrl;
            isImageLoaded = true;
        };
    }

    $: if (!isImageLoaded) {
        loadImage(imageUrl);
    }

    function handleWheel(event: WheelEvent) {
        const container = event.currentTarget as HTMLElement;
        if (event.deltaX !== 0) {
            return;
        }
        if (event.deltaY > 0) {
            container.scrollLeft += 50;
        } else {
            container.scrollLeft -= 50;
        }
        event.preventDefault();
    }
</script>

<style>
    .card {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        padding: 24px 24px 32px;
        gap: 24px;
        width: 360px;
        background: #FFF;
        box-shadow: 0 4px 16px rgba(0 0 0 / 7%);
        border-radius: 32px;
    }

    .image-container {
        width: 100%;
        height: 250px;
        position: relative;
        max-height: calc(100vh - 100px);
    }

    .label {
        position: absolute;
        left: 18px;
        top: 15px;
        font-weight: 400;
        font-size: 24px;
        line-height: 29px;
        color: #FFF;
        text-transform: capitalize;
        text-shadow: 1px 1px 2px #333;
    }

    .location {
        font-weight: 400;
        font-size: 20px;
        line-height: 24px;
        color: #9B9B9B;
        margin: 0;
    }

    .title {
        font-weight: 600;
        font-size: 32px;
        line-height: 39px;
        color: #2F2D2C;
        margin: 0;
    }

    .subtitle {
        font-weight: 400;
        font-size: 24px;
        line-height: 29px;
        color: #2F2D2C;
        margin: 0;
    }

    .tags {
        display: flex;
        flex-flow: row nowrap;
        gap: 8px;
        width: 100%;
        overflow-x: auto;
        -ms-overflow-style: none;
        scrollbar-width: none;
    }

    .tags::-webkit-scrollbar {
        display: none;
    }

    .tag {
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 8px 16px;
        border-radius: 8px;
        font-weight: 400;
        font-size: 16px;
        line-height: 19px;
        color: #2F2D2C;
        white-space: nowrap;
        text-shadow: 1px 1px 2px #FFF;
    }

    .coffee-image {
        width: 100%;
        height: 100%;
        object-fit: cover;
        border-radius: 16px;
        background-size: cover;
    }

    @media (width < 400px) {
        .card {
            padding: 0;
            overflow: hidden;
        }

        .content {
            padding: 10px;
        }

        .tags {
            padding: 15px 5px;
        }
    }

    @media (width <= 320px) {
        .card {
            width: 100%;
            padding: 0;
        }

        .coffee-image {
            border-radius: 0;
        }

        .title {
            font-size: 24px;
            line-height: 29px;
        }

        .subtitle {
            font-size: 18px;
            line-height: 22px;
        }

        .tags {
            gap: 4px;
        }

        .tag {
            font-size: 14px;
            padding: 4px 8px;
        }
    }
</style>

<div class="card">
    <div class="image-container">
        <div class="placeholder" style:display={!imageLoaded ? 'block' : 'none'}></div>
        <img
                src={imageUrl}
                alt="Coffee"
                class="coffee-image"
                loading="lazy"
        >
        <div class="label">{coffee.intensifier}</div>
    </div>
    <div class="content">
        <div class="location">{coffee.origin}</div>
        <div class="title">{coffee.blend_name}</div>
        <div class="subtitle">{coffee.variety}</div>
    </div>
    <div class="tags" on:wheel={handleWheel}>
        {#each coffee.notes.split(', ') as note}
            <div class="tag" style="background-color: {getTagColor(note)};">{note}</div>
        {/each}
    </div>
</div>