document.addEventListener('DOMContentLoaded', function() {
    // --- Mobile Menu Toggle ---
    const mobileMenuButton = document.getElementById('mobile-menu-button');
    const mobileMenu = document.getElementById('mobile-menu');
    mobileMenuButton.addEventListener('click', () => {
        mobileMenu.classList.toggle('hidden');
    });
    
    mobileMenu.addEventListener('click', (e) => {
        if (e.target.tagName === 'A') {
            mobileMenu.classList.add('hidden');
        }
    });

    // --- Header shadow on scroll ---
    const header = document.getElementById('header');
    window.addEventListener('scroll', () => {
        if (window.scrollY > 50) {
            header.classList.add('shadow-lg');
        } else {
            header.classList.remove('shadow-lg');
        }
    });

    // --- Fade-in sections on scroll ---
    const sections = document.querySelectorAll('.fade-in-section');
    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                entry.target.classList.add('is-visible');
            }
        });
    }, { threshold: 0.1 });

    sections.forEach(section => {
        observer.observe(section);
    });

    // --- Portfolio Modal & Slideshow Logic ---
    const modal = document.getElementById('portfolio-modal');
    const modalClose = document.getElementById('modal-close');
    const portfolioItems = document.querySelectorAll('.portfolio-item');

    const modalTitle = document.getElementById('modal-title');
    const modalType = document.getElementById('modal-type');
    const modalDesc = document.getElementById('modal-desc');
    const modalToolsContainer = document.getElementById('modal-tools');

    const slideshow = document.getElementById('modal-slideshow');
    const slidesContainer = document.getElementById('slides-container');
    const prevButton = document.getElementById('prev-slide');
    const nextButton = document.getElementById('next-slide');
    const dotsContainer = document.getElementById('slide-dots');

    let currentSlide = 0;
    let slides = [];
    let dots = [];

    function showSlide(n) {
        // Pause all videos first, just in case
        slides.forEach(slide => {
            const video = slide.querySelector('video');
            if (video) video.pause();
        });
        
        // Wrap around logic for current slide index
        if (n >= slides.length) {
            currentSlide = 0;
        } else if (n < 0) {
            currentSlide = slides.length - 1;
        } else {
            currentSlide = n;
        }

        // Update active class for slides
        slides.forEach((slide, index) => {
            slide.classList.toggle('active', index === currentSlide);
        });
        // Update active class for dots
        dots.forEach((dot, index) => {
            dot.classList.toggle('bg-indigo-500', index === currentSlide);
            dot.classList.toggle('bg-gray-500', index !== currentSlide);
        });
        
        // If the new active slide is a video, try to play it
        const activeSlide = slides[currentSlide];
        const activeVideo = activeSlide.querySelector('video');
        if (activeVideo) {
            activeVideo.currentTime = 0;
            const playPromise = activeVideo.play();
            if (playPromise !== undefined) {
                playPromise.catch(error => {
                    // Autoplay was prevented. This is a common browser policy.
                    if (error.name !== 'AbortError') {
                        console.error('Video playback error:', error);
                    }
                });
            }
        }
    }

    function changeSlide(direction) {
        showSlide(currentSlide + direction);
    }
    
    // [FIXED] Updated the way slides are created for robustness
    const openModal = (item) => {
        // Get data from attributes
        const title = item.getAttribute('data-title');
        const type = item.getAttribute('data-type');
        const tools = item.getAttribute('data-tools').split(',').map(t => t.trim());
        const desc = item.getAttribute('data-desc');
        const mediaData = JSON.parse(item.getAttribute('data-media') || '[]');

        // Populate text content
        modalTitle.textContent = title;
        modalType.textContent = type;
        modalDesc.textContent = desc;
        
        // Populate tools as tags
        modalToolsContainer.innerHTML = '';
        tools.forEach(tool => {
            const toolTag = document.createElement('span');
            toolTag.className = 'bg-gray-700 text-indigo-300 text-sm font-medium px-3 py-1 rounded-full';
            toolTag.textContent = tool;
            modalToolsContainer.appendChild(toolTag);
        });

        // Clear previous slides and dots
        slidesContainer.innerHTML = '';
        dotsContainer.innerHTML = '';
        slides = [];
        dots = [];

        // Create and populate slideshow
        mediaData.forEach((media, index) => {
            // Create slide element
            const slideEl = document.createElement('div');
            slideEl.className = 'slide flex items-center justify-center bg-black';
            
            if (media.type === 'video') {
                const video = document.createElement('video');
                video.src = media.src;
                video.className = 'max-w-full max-h-full';
                video.controls = true;
                video.loop = true;
                video.muted = true;
                video.playsInline = true; // For iOS compatibility
                slideEl.appendChild(video);
            } else { // 'image'
                const img = document.createElement('img');
                img.src = media.src;
                img.alt = `${title} - media ${index + 1}`;
                img.className = 'max-w-full max-h-full object-contain';
                img.onerror = function() {
                    this.onerror = null;
                    this.src = `https://placehold.co/1280x720/111827/d1d5db?text=Image+Not+Found`;
                };
                slideEl.appendChild(img);
            }
            slidesContainer.appendChild(slideEl);
            slides.push(slideEl);
            
            // Create dot element
            const dotEl = document.createElement('button');
            dotEl.className = 'dot w-3 h-3 rounded-full bg-gray-500 hover:bg-indigo-400';
            dotEl.addEventListener('click', () => showSlide(index));
            dotsContainer.appendChild(dotEl);
            dots.push(dotEl);
        });

        // Show/hide nav based on slide count
        const showNav = mediaData.length > 1;
        prevButton.style.display = showNav ? 'block' : 'none';
        nextButton.style.display = showNav ? 'block' : 'none';
        dotsContainer.style.display = showNav ? 'flex' : 'none';

        // Show modal and first slide
        modal.classList.remove('hidden');
        document.body.style.overflow = 'hidden';
        showSlide(0);
    };

    portfolioItems.forEach(item => {
        item.addEventListener('click', () => openModal(item));
    });

    const closeModal = () => {
        modal.classList.add('hidden');
        document.body.style.overflow = '';
        // Stop all videos when closing the modal
        slides.forEach(slide => {
            const video = slide.querySelector('video');
            if (video) video.pause();
        });
    };

    // Event listeners for closing/navigation
    modalClose.addEventListener('click', closeModal);
    modal.addEventListener('click', (e) => {
        if (e.target === modal) closeModal();
    });
    document.addEventListener('keydown', (e) => {
        if (modal.classList.contains('hidden')) return;
        if (e.key === "Escape") closeModal();
        if (e.key === "ArrowLeft") changeSlide(-1);
        if (e.key === "ArrowRight") changeSlide(1);
    });
    prevButton.addEventListener('click', () => changeSlide(-1));
    nextButton.addEventListener('click', () => changeSlide(1));
});