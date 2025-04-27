<script setup lang="ts">
import { Head } from '@inertiajs/vue3';
import { onMounted, ref } from 'vue';
import { gsap } from 'gsap';
import { TextPlugin } from 'gsap/TextPlugin'
import ProjectCard from '@/components/app/ProjectCard.vue';
import Cursor from '@/components/app/Cursor.vue';

const cursorComponent = ref<{ cursor: HTMLElement | null } | null>(null)

gsap.registerPlugin(TextPlugin)

const loader = ref<HTMLElement | null>(null)
const main = ref<HTMLElement | null>(null)
const introText = ref<HTMLElement | null>(null)
const progressBar = ref<HTMLElement | null>(null)
const spinnerImg = ref<HTMLImageElement | null>(null)
const textElement = ref<HTMLElement | null>(null)
const rotatingImg = ref(null)
let rotationTween = null
let isSpinning = false

//intro text animation
const isPaused = ref(false)
let typingTween: gsap.core.Tween | null = null
let delayedCall: gsap.core.DelayedCall | null = null

const greetings = [
  "Hello,",       // English
  "မင်္ဂလာပါ,",     // Burmese
  "مرحبًا, ",       // Arabic
  "Namaste, ",     // Hindi
]

let index = 0

function animateText() {
  if (!textElement.value) return;

  const nextText = greetings[index];
  index = (index + 1) % greetings.length;

  // Clear current content
  textElement.value.textContent = ''

  typingTween = gsap.to(textElement.value, {
    duration: 1,
    text: nextText,
    opacity: 1,
    ease: 'none',
    onComplete: () => {
      // Wait 2s then fade out and call animateText again
      delayedCall = gsap.delayedCall(2, () => {
        gsap.to(textElement.value, {
          duration: 1,
          opacity: 0,
          onComplete: () => {
            if (!isPaused.value) {
              animateText()
            }
          }
        });
      });
    }
  });
}

const projects = [
  {
    title: 'EduPlus Admin ( Backend )',
    image: '/storage/project-imgs/eduplus-admin-dashboard.png',
    link: 'https://www.eduplusmyanmar.com/',
    description: "EduPlus is a full-featured School and LMS platform. It supports course enrollments by batches and subjects, detailed attendance tracking for physical and online classes, and flexible payment options including monthly, one-time, and custom fees. Mostly handled change requests and bug fixes. Developed a new feature for the system which is the dynamic Report Card module to help schools easily manage student assessments.",
    stack: ['php', 'laravel', 'bootstrap' ,'mysql'],
  },
  {
    title: 'EduPlus ( Frontend )',
    image: '/storage/project-imgs/eduplus-fe-home.png',
    description: 'Frontend application for students, teachers and guardians. Handled change requests from small adjustments to flow changes of the whole features and bug fixes.',
    stack: ['javascript', 'vue', 'tailwind'],

  },
  {
    title: 'Thazin ( CMS )',
    image: '/storage/project-imgs/thazin-jewelry.png',
    link: 'https://thazinjewelry.com/',
    description: "Website for Thazin Jewellry Shop. I contributed in developing front-end for this website including all the pages and responsive of the pages.",
    stack: ['php', 'laravel', 'tailwind', 'filament'],
  },
  {
    title: 'Artist Runorl Gallery ( CMS )',
    image: '/storage/project-imgs/runorl-gallery.png',
    link: 'https://www.runorl-gallery.art/',
    description: 'Website for my Dad who is an artist. I wanted to make a full stack app which is useful in real-life and have a positive impact to people around me so I developed this full stack CMS website. ',
    stack: ['php', 'laravel', 'bootstrap','mysql'],
  },
  {
    title: 'Honyx Designs Showcase',
    image: '/storage/project-imgs/fe-sft.png',
    link: 'https://front-end-project-sft.vercel.app/',
    description: "Personal frontend Project for practice.",
    stack: ['html', 'css', 'bootstrap','javascript'],
  },
  
];

onMounted(() => {
    // Prepare the tween (but paused initially)
    rotationTween = gsap.to(rotatingImg.value, {
        rotation: '+=360',
        duration: 5,
        ease: 'linear',
        repeat: -1,
        paused: true,
        transformOrigin: '50% 50%',
    })
    // Add click toggle
    rotatingImg.value.addEventListener('click', () => {
        isSpinning = !isSpinning
        if (isSpinning) {
        rotationTween.resume()
        } else {
        rotationTween.pause()
        }
    })

    if (spinnerImg.value) {
        gsap.to(spinnerImg.value, {
            rotation: 360,
            duration: 10,
            repeat: -1,
            ease: 'linear',
            transformOrigin: '50% 50%',
        })
    }

    //Fake loading then hide loader
    const tl = gsap.timeline();
    tl.to(loader.value, {
        delay: 2,
        opacity: 0,
        duration: 0.2,
        onComplete: () => {
        if (loader.value) loader.value.style.display = 'none'
        }
    })

    //Fade in main
    tl.to(main.value, {
        opacity: 1,
        duration: 0.1
    })

    //Animate intro text block
    tl.from(introText.value, {
        y: 60,
        opacity: 0,
        duration: 1.2,
        ease: 'power4.out'
    })

    //Animate letters individually
    const spans = introText.value?.querySelectorAll('span') ?? []
    tl.from(spans, {
        opacity: 0,
        x: 50,
        stagger: 0.05,
        duration: 3,
        ease: 'back.out(1.7)'
    })

    //intro text animation
    if (textElement.value) {
        textElement.value.textContent = ''
        gsap.to(textElement.value, { duration: 1, opacity: 1 })

        // Add click to pause/resume
        textElement.value.addEventListener('click', () => {
            isPaused.value = !isPaused.value

            if (isPaused.value) {
            typingTween?.pause()
            delayedCall?.pause()
            } else {
            typingTween?.resume()
            delayedCall?.resume()
            }
        });
    }
    // Progress bar
    const progress = { value: 0 }
    gsap.to(progress, {
        value: 100,
        duration: 1.8,
        ease: 'linear',
        onUpdate: () => {
        if (progressBar.value) {
            progressBar.value.style.width = `${progress.value}%`
        }
        }
    })

    animateText()

    const cursor = cursorComponent.value?.cursor
    if (!cursor) return

    const elements = [...document.querySelectorAll('a'), ...document.querySelectorAll('.interactive')]
    
    elements.forEach(el => {
        el.addEventListener('mouseenter', () => {
            gsap.to(cursor, {
                scale: 1.5,
                duration: 0.8,
                rotate: 580,
                ease: 'power2.out'
            })
        })
        el.addEventListener('mouseleave', () => {
            gsap.to(cursor, {
                scale: 1,
                duration: 0.6,
                rotate: 0,
                ease: 'power2.out'
            })
        })
    })

    window.addEventListener('click', () => {
        gsap.fromTo(cursor,
            { rotation: 0 },
            {
            rotation: 360,
            duration: 0.6,
            ease: 'power2.out',
            }
        )
    })
})

</script>

<style scoped>
#textElement {
    cursor: none;
}

</style>

<template>
    <Head title="Ye Wint Pai | Portfolio">
        <link rel="preconnect" href="https://rsms.me/" />
        <link rel="stylesheet" href="https://rsms.me/inter/inter.css" />
    </Head>

    <!-- loading screen -->
    <div ref="loader" class="fixed inset-0 z-50 bg-sky-100 flex items-center justify-center">
        <div>
            <img 
                :src="`/storage/images/circle-with-stars.png`"
                class="mb-16"
                ref="spinnerImg" 
                alt="loading-spinner" width="150px" height="150px">
        </div>
        <h1 class="text-4xl text-cyan-700 font-bold animate-pulse">Just a sec...</h1>

        <div class="absolute mt-24 w-1/2 h-2 bg-cyan-300 rounded-full overflow-hidden">
            <div ref="progressBar" class="h-full bg-cyan-700 w-0"></div>
        </div>
    </div>

    <Cursor ref="cursorComponent" />

    <div class="md:w-5/6 w-full mx-auto bg-sky-100 text-cyan-700 py-12" id="background">
        <div class="mt-18" id="hero">
            <div class="flex md:flex-row flex-col-reverse justify-between md:items-start items-center md:px-16 px:0">
                <div class="flex flex-col items-around text-start">
                    <div class="flex my-auto">
                        <span id="textElement" ref="textElement" class="text-5xl font-bold interactive">Hello</span>
                        <span class="text-5xl font-bold">&nbsp;I'm </span>
                    </div>
                    <span class="text-5xl my-1 font-extrabold">Ye Wint Pai Myel Aung</span>

                    <div class="flex md:justify-start justify-center items-start  mt-4">
                        <a href="https://github.com/YeWintPai" target="_blank" class="bg-cyan-700 hover:bg-cyan-500 text-xs text-white font-bold me-3 py-2 px-3 rounded">GitHub</a>
                        <a href="https://www.linkedin.com/in/ye-wint-pai-myel-aung-9269662bb/" target="_blank" class="bg-cyan-700 hover:bg-cyan-500 text-xs text-white font-bold mx-3 py-2 px-4 rounded">LinkedIn</a>
                        <a href="" target="_blank" class="bg-cyan-700 hover:bg-cyan-500 text-xs text-white font-bold mx-3 py-2 px-4 rounded">CV</a>
                    </div>
                </div>
                
                <div class="">
                    <img 
                    ref="rotatingImg"
                    :src="`/storage/images/circle-with-stars.png`" 
                    class="size-40 object-contain interactive"
                    alt="circle-with-stars"
                    >
                </div>
            </div>

            <div class="md:px-16 px-0 mt-12">
                <p class="md:text-2xl text-lg md:text-start text-center">
                    A passionate full stack developer with a year of hands-on experience in PHP, Laravel, and Vue.js.
                    I have a strong backend foundation but also enjoy crafting intuitive user interfaces with Tailwind CSS, JavaScript, and exploring new frontend tools like React.
                    I'm currently based in Dubai, UAE, looking for exciting opportunities where I can bring value, grow with the team, and work on meaningful products.
                </p>
            </div>

        </div>

        <div class="md:px-16 px-0 mt-12" id="experiences">
            <h1 class="md:text-start text-center text-4xl font-extrabold">Experiences</h1>

            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 px-4 py-10 items-start">

                <ProjectCard
                    v-for="(project, index) in projects"
                    :key="index"
                    :title="project.title"
                    :image="project.image"
                    :link="project.link"
                    :description="project.description"
                    :stack="project.stack"
                />
            </div>


        </div>

        <div class="md:px-16 px-0 mt-12 pb-12" id="skills">
            <h1 class="md:text-start text-center text-4xl font-extrabold">Skills</h1>
            
            <div class="mt-6">
                <h1 class="text-2xl text-center font-bold">Frontend</h1>
                <div class="flex flex-wrap justify-center md:gap-4 gap-6 mt-2">
            
                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-html5"></i>
                        </span>
                        <span class="text-black">Html</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-css3-alt"></i>
                        </span>
                        <span class="text-black">Css</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-js"></i>
                        </span>
                        <span class="text-black">JavaScript</span>
                    </div>
                    
                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-bootstrap"></i>
                        </span>
                        <span class="text-black">Bootstrap</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <svg width="46px" height="46px" viewBox="0 0 32 32" xmlns="http://www.w3.org/2000/svg" fill="#000000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"><title>file_type_tailwind</title><path d="M9,13.7q1.4-5.6,7-5.6c5.6,0,6.3,4.2,9.1,4.9q2.8.7,4.9-2.1-1.4,5.6-7,5.6c-5.6,0-6.3-4.2-9.1-4.9Q11.1,10.9,9,13.7ZM2,22.1q1.4-5.6,7-5.6c5.6,0,6.3,4.2,9.1,4.9q2.8.7,4.9-2.1-1.4,5.6-7,5.6c-5.6,0-6.3-4.2-9.1-4.9Q4.1,19.3,2,22.1Z" style="fill:#0e7490"></path></g></svg>
                        </span>
                        <span class="text-black">Tailwind</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-vuejs"></i>
                        </span>
                        <span class="text-black">Vue</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-react"></i>
                        </span>
                        <span class="text-black">React</span>
                    </div>
                </div>
            </div>

            <div class="mt-6">
                <h1 class="text-2xl text-center font-bold">Backend</h1>
                <div class="flex flex-wrap justify-center md:gap-4 gap-6 mt-2">
            
                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-php"></i>
                        </span>
                        <span class="text-black">Php</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-laravel"></i>
                        </span>
                        <span class="text-black">Laravel</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-node"></i>
                        </span>
                        <span class="text-black">Node</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <svg width="46px" height="46px" viewBox="0 0 32 32" xmlns="http://www.w3.org/2000/svg" fill="#000000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"><title>file_type_node2</title><path d="M16,2A14,14,0,1,1,2,16,14,14,0,0,1,16,2Z" style="fill:#0e7490;fill-rule:evenodd"></path><path d="M16,30a2.151,2.151,0,0,1-1.076-.288L11.5,27.685c-.511-.286-.262-.387-.093-.446a6.828,6.828,0,0,0,1.549-.7.263.263,0,0,1,.255.019l2.631,1.563a.34.34,0,0,0,.318,0l10.26-5.922a.323.323,0,0,0,.157-.278V10.075a.331.331,0,0,0-.159-.283L16.158,3.875a.323.323,0,0,0-.317,0L5.587,9.794a.33.33,0,0,0-.162.281V21.916a.315.315,0,0,0,.161.274L8.4,23.814c1.525.762,2.459-.136,2.459-1.038V11.085a.3.3,0,0,1,.3-.3h1.3a.3.3,0,0,1,.3.3V22.777c0,2.035-1.108,3.2-3.038,3.2a4.389,4.389,0,0,1-2.363-.642L4.661,23.788a2.166,2.166,0,0,1-1.076-1.872V10.075A2.162,2.162,0,0,1,4.661,8.2L14.922,2.276a2.246,2.246,0,0,1,2.156,0L27.338,8.2a2.165,2.165,0,0,1,1.077,1.87V21.916a2.171,2.171,0,0,1-1.077,1.872l-10.26,5.924A2.152,2.152,0,0,1,16,30Zm3.488-8.257c3.251,0,5.115-1.28,5.115-3.516,0-2.216-1.5-2.807-4.651-3.223-3.186-.422-3.51-.639-3.51-1.385,0-.616.274-1.438,2.634-1.438,2.108,0,2.885.454,3.2,1.875a.3.3,0,0,0,.288.232H23.9a.3.3,0,0,0,.295-.323c-.206-2.448-1.832-3.589-5.12-3.589-2.925,0-4.67,1.235-4.67,3.305,0,2.246,1.736,2.866,4.544,3.144,3.359.329,3.62.82,3.62,1.481,0,1.147-.92,1.636-3.082,1.636-2.715,0-3.313-.682-3.513-2.032a.3.3,0,0,0-.295-.251H14.351a.3.3,0,0,0-.3.3C14.054,19.682,14.995,21.743,19.485,21.743Z" style="fill:#cffafe"></path></g></svg>
                        </span>
                        <span class="text-black">Express</span>
                    </div>
                </div>
            </div>

            <div class="mt-6">
                <h1 class="text-2xl text-center font-bold">Others</h1>
                <div class="flex flex-wrap justify-center md:gap-4 gap-6 mt-2">
            
                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <svg fill="#0e7490" width="46px" height="46px" viewBox="0 0 32 32" version="1.1" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <title>mysql</title> <path d="M30.026 15.139c-0.082-0.006-0.177-0.010-0.273-0.010-0.484 0-0.947 0.090-1.373 0.254l0.026-0.009c-0.125 0.050-0.325 0.050-0.342 0.209 0.069 0.066 0.079 0.175 0.137 0.267 0.117 0.198 0.261 0.366 0.429 0.506l0.003 0.003c0.175 0.137 0.35 0.27 0.534 0.387 0.325 0.2 0.694 0.319 1.012 0.52 0.181 0.117 0.366 0.266 0.55 0.391 0.091 0.062 0.15 0.175 0.267 0.215v-0.025c-0.057-0.075-0.075-0.184-0.131-0.267-0.084-0.084-0.167-0.159-0.25-0.241-0.248-0.325-0.535-0.603-0.857-0.835l-0.012-0.008c-0.267-0.182-0.852-0.437-0.962-0.744l-0.016-0.018c0.218-0.031 0.412-0.077 0.599-0.139l-0.024 0.007c0.284-0.075 0.544-0.059 0.837-0.132 0.132-0.034 0.266-0.075 0.4-0.117v-0.075c-0.15-0.15-0.262-0.354-0.417-0.494-0.409-0.36-0.86-0.698-1.335-1.002l-0.045-0.027c-0.262-0.167-0.595-0.275-0.871-0.417-0.1-0.050-0.267-0.075-0.325-0.159-0.13-0.185-0.245-0.397-0.336-0.621l-0.008-0.022q-0.368-0.714-0.684-1.453c-0.144-0.383-0.287-0.697-0.448-1.001l0.023 0.047c-0.786-1.319-1.881-2.379-3.188-3.102l-0.043-0.022c-0.309-0.153-0.668-0.272-1.045-0.339l-0.025-0.004c-0.209-0.010-0.417-0.025-0.625-0.034-0.146-0.094-0.272-0.19-0.39-0.296l0.003 0.003c-0.475-0.3-1.704-0.95-2.054-0.090-0.225 0.542 0.334 1.077 0.527 1.352 0.154 0.183 0.294 0.388 0.415 0.605l0.010 0.020c0.059 0.145 0.075 0.294 0.134 0.445 0.145 0.452 0.292 0.823 0.459 1.182l-0.026-0.062c0.099 0.199 0.202 0.368 0.317 0.528l-0.008-0.012c0.067 0.091 0.182 0.134 0.209 0.284-0.086 0.181-0.153 0.391-0.19 0.61l-0.002 0.014c-0.108 0.332-0.171 0.715-0.171 1.112 0 0.621 0.153 1.206 0.423 1.72l-0.010-0.020c0.134 0.207 0.452 0.667 0.878 0.491 0.375-0.15 0.292-0.625 0.4-1.043 0.025-0.1 0.009-0.166 0.060-0.234v0.019c0.117 0.235 0.235 0.459 0.342 0.694 0.302 0.435 0.661 0.805 1.071 1.11l0.013 0.009c0.2 0.15 0.359 0.41 0.609 0.502v-0.025h-0.019c-0.057-0.062-0.12-0.117-0.189-0.164l-0.004-0.002c-0.155-0.152-0.299-0.316-0.429-0.489l-0.008-0.011c-0.326-0.44-0.636-0.938-0.905-1.461l-0.029-0.061c-0.137-0.262-0.252-0.545-0.362-0.804-0.050-0.1-0.050-0.25-0.134-0.3-0.148 0.166-0.281 0.351-0.392 0.55l-0.008 0.016c-0.128 0.373-0.212 0.804-0.234 1.251l-0 0.011c-0.034 0.009-0.017 0-0.034 0.018-0.267-0.065-0.359-0.342-0.459-0.575-0.136-0.366-0.215-0.79-0.215-1.231 0-0.356 0.051-0.7 0.147-1.025l-0.006 0.026c0.059-0.175 0.309-0.727 0.209-0.895-0.052-0.159-0.217-0.25-0.309-0.379-0.109-0.154-0.209-0.329-0.292-0.514l-0.008-0.020c-0.2-0.467-0.3-0.985-0.517-1.452-0.131-0.244-0.269-0.454-0.424-0.65l0.007 0.009c-0.165-0.191-0.317-0.404-0.449-0.63l-0.011-0.020c-0.041-0.091-0.1-0.242-0.034-0.342 0.012-0.058 0.058-0.103 0.117-0.112l0.001-0c0.11-0.090 0.419 0.027 0.527 0.077 0.317 0.12 0.59 0.261 0.843 0.427l-0.016-0.010c0.117 0.082 0.244 0.241 0.394 0.282h0.175c0.267 0.059 0.569 0.018 0.819 0.091 0.459 0.155 0.856 0.349 1.223 0.587l-0.021-0.013c1.104 0.713 1.988 1.677 2.586 2.816l0.020 0.041c0.1 0.192 0.144 0.369 0.235 0.569 0.175 0.412 0.391 0.829 0.569 1.227 0.169 0.428 0.369 0.798 0.607 1.139l-0.012-0.018c0.125 0.175 0.627 0.266 0.852 0.357 0.237 0.083 0.427 0.162 0.611 0.251l-0.037-0.016c0.287 0.175 0.567 0.375 0.837 0.567 0.137 0.095 0.554 0.304 0.579 0.472zM18.302 22.452c0 0.015 0.001 0.032 0.001 0.049 0 0.558-0.249 1.057-0.643 1.393l-0.003 0.002c-0.432 0.352-0.989 0.566-1.596 0.566-0.047 0-0.094-0.001-0.14-0.004l0.006 0c-0.739-0.010-1.419-0.25-1.976-0.651l0.010 0.007 0.296-0.595c0.429 0.24 0.939 0.389 1.481 0.41l0.006 0c0.027 0.002 0.058 0.003 0.090 0.003 0.332 0 0.641-0.104 0.894-0.281l-0.005 0.003c0.229-0.174 0.375-0.446 0.375-0.752 0-0.006-0-0.011-0-0.017v0.001c0-0.412-0.287-0.762-0.81-1.056-0.485-0.266-1.453-0.821-1.453-0.821-0.478-0.296-0.791-0.817-0.791-1.411 0-0.021 0-0.042 0.001-0.063l-0 0.003c-0.001-0.019-0.001-0.041-0.001-0.063 0-0.515 0.227-0.977 0.586-1.291l0.002-0.002c0.382-0.324 0.881-0.521 1.426-0.521 0.035 0 0.069 0.001 0.103 0.002l-0.005-0c0.009-0 0.020-0 0.031-0 0.639 0 1.234 0.191 1.73 0.52l-0.012-0.007-0.266 0.595c-0.391-0.176-0.847-0.282-1.327-0.287l-0.002-0c-0.024-0.002-0.051-0.003-0.079-0.003-0.28 0-0.538 0.098-0.74 0.262l0.002-0.002c-0.189 0.157-0.309 0.392-0.31 0.655v0c0 0.41 0.292 0.762 0.832 1.062 0.491 0.269 1.483 0.837 1.483 0.837 0.488 0.287 0.811 0.809 0.811 1.407 0 0.018-0 0.037-0.001 0.055l0-0.003zM20.374 22.983c-0.273-0.545-0.432-1.187-0.432-1.866 0-0.107 0.004-0.213 0.012-0.317l-0.001 0.014q0-2.611 1.587-2.612c0.026-0.002 0.057-0.003 0.089-0.003 0.475 0 0.892 0.248 1.129 0.622l0.003 0.005c0.271 0.542 0.43 1.182 0.43 1.858 0 0.104-0.004 0.207-0.011 0.309l0.001-0.014q0 2.632-1.587 2.634c-0.027 0.002-0.058 0.003-0.089 0.003-0.475 0-0.893-0.248-1.13-0.622l-0.003-0.005zM24.488 24.535l-1.27-0.625c0.116-0.097 0.22-0.199 0.316-0.309l0.003-0.003c0.513-0.692 0.821-1.563 0.821-2.505 0-0.109-0.004-0.217-0.012-0.324l0.001 0.014q0-3.43-2.693-3.432c-0.040-0.002-0.087-0.003-0.134-0.003-0.767 0-1.456 0.337-1.925 0.872l-0.002 0.003c-0.511 0.692-0.818 1.562-0.818 2.504 0 0.106 0.004 0.211 0.012 0.315l-0.001-0.014c-0.009 0.101-0.014 0.219-0.014 0.338 0 0.874 0.274 1.684 0.74 2.349l-0.009-0.013c0.449 0.478 1.086 0.776 1.791 0.776 0.066 0 0.131-0.003 0.195-0.008l-0.009 0.001c0.009 0 0.021 0 0.032 0 0.311 0 0.612-0.045 0.897-0.128l-0.022 0.006 1.656 0.965 0.45-0.777zM28.636 24.366h-3.287v-6.91h1.106v6.061h2.181zM13.235 19.268c-0.287 2.084-0.944 3.965-1.905 5.65l0.040-0.077c-0.385 0.741-1.113 1.257-1.968 1.34l-0.010 0.001c-0.259-0.014-0.5-0.076-0.719-0.177l0.012 0.005v-0.617c0.137 0.021 0.295 0.033 0.456 0.033 0.009 0 0.018-0 0.028-0h-0.001c0.016 0.001 0.034 0.001 0.052 0.001 0.289 0 0.554-0.105 0.758-0.28l-0.002 0.001c0.22-0.181 0.361-0.451 0.369-0.755l0-0.001c-0.053-0.438-0.154-0.837-0.299-1.214l0.012 0.034-1.267-3.944h1.137l0.909 2.949c0.162 0.416 0.256 0.898 0.256 1.401 0 0.001 0 0.002 0 0.002v-0c0.482-1.262 0.848-2.734 1.034-4.261l0.009-0.092zM8.215 24.366h-1.158q-0.049-2.761-0.337-5.511h-0.010l-1.762 5.511h-0.881l-1.75-5.511h-0.012q-0.205 2.751-0.244 5.511h-1.056q0.103-3.685 0.512-6.911h1.437l1.668 5.079h0.010l1.683-5.079h1.368q0.454 3.777 0.535 6.911zM21.505 7.879c-0.002 0-0.005-0-0.008-0-0.119 0-0.234 0.015-0.344 0.043l0.010-0.002v0.016h0.017c0.086 0.128 0.174 0.239 0.269 0.343l-0.002-0.002c0.067 0.134 0.125 0.267 0.192 0.4l0.017-0.019c0.109-0.086 0.178-0.218 0.178-0.366 0-0.018-0.001-0.035-0.003-0.053l0 0.002c-0.050-0.059-0.057-0.117-0.1-0.175-0.050-0.084-0.157-0.125-0.225-0.191z"></path> </g></svg>
                        </span>
                        <span class="text-black">MySQL</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <svg fill="#0e7490" width="46px" height="46px" viewBox="0 0 32 32" version="1.1" xmlns="http://www.w3.org/2000/svg" stroke="#0e7490" stroke-width="0.00032"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <title>mongodb</title> <path d="M15.821 23.185s0-10.361 0.344-10.36c0.266 0 0.612 13.365 0.612 13.365-0.476-0.056-0.956-2.199-0.956-3.005zM22.489 12.945c-0.919-4.016-2.932-7.469-5.708-10.134l-0.007-0.006c-0.338-0.516-0.647-1.108-0.895-1.732l-0.024-0.068c0.001 0.020 0.001 0.044 0.001 0.068 0 0.565-0.253 1.070-0.652 1.409l-0.003 0.002c-3.574 3.034-5.848 7.505-5.923 12.508l-0 0.013c-0.001 0.062-0.001 0.135-0.001 0.208 0 4.957 2.385 9.357 6.070 12.115l0.039 0.028 0.087 0.063q0.241 1.784 0.412 3.576h0.601c0.166-1.491 0.39-2.796 0.683-4.076l-0.046 0.239c0.396-0.275 0.742-0.56 1.065-0.869l-0.003 0.003c2.801-2.597 4.549-6.297 4.549-10.404 0-0.061-0-0.121-0.001-0.182l0 0.009c-0.003-0.981-0.092-1.94-0.261-2.871l0.015 0.099z"></path> </g></svg>
                        </span>
                        <span class="text-black">MongoDB</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-git-alt"></i>
                        </span>
                        <span class="text-black">Git</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <i class="fa-brands fa-github"></i>
                        </span>
                        <span class="text-black">GitHub</span>
                    </div>

                    <div class="flex flex-col justify-center items-center md:w-[100px] w-[70px] md:h-[100px] h-[70px] hover:scale-[1.02] hover:shadow-sm transition-all duration-200 bg-cyan-100 rounded-sm">
                        <span class="md:text-6xl text-3xl text-cyan-700">
                            <svg fill="#0e7490" width="46px" height="46px" viewBox="0 0 32 32" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M18.036 0.131c-8.765-1.12-16.781 5.067-17.905 13.833-1.12 8.765 5.067 16.781 13.833 17.905 8.765 1.12 16.781-5.067 17.9-13.833 1.125-8.765-5.061-16.781-13.828-17.905zM21.328 10.115c-0.297 0-0.579 0.12-0.787 0.333l-5.937 5.932-0.411-0.407-0.855-0.859c5.849-5.833 6.907-5.891 7.989-5zM14.849 16.593l5.916-5.921c0.328-0.344 0.875-0.339 1.204 0.005 0.323 0.349 0.291 0.896-0.073 1.197l-6.265 5.5zM15.287 17.521l-1.469 0.317c-0.031 0.005-0.072-0.011-0.088-0.047-0.016-0.032-0.011-0.068 0.016-0.095l0.859-0.859zM11.547 16.907l1.568-1.563 1.172 1.172-2.641 0.567c-0.047 0.011-0.093-0.009-0.115-0.052-0.025-0.041-0.015-0.093 0.016-0.124zM6.688 24.984c-0.057-0.005-0.1-0.057-0.095-0.109 0.005-0.025 0.016-0.047 0.032-0.063h0.005l1.26-1.26 1.631 1.631zM9.921 23.307c-0.124 0.068-0.187 0.209-0.156 0.344l0.271 1.152c0.043 0.167-0.161 0.28-0.281 0.156h-0.005l-1.635-1.636 5.016-5.011 2.427-0.525 1.161 1.167c-1.672 1.468-3.959 2.932-6.797 4.353zM16.959 18.74l-1.12-1.12 6.265-5.5c0.057-0.052 0.109-0.109 0.156-0.167-0.192 1.792-2.703 4.323-5.301 6.787zM21.839 10.125h-0.005c-2.183-2.193 0.901-5.563 3.276-3.584l-2.145 2.152c-0.063 0.061-0.063 0.167 0 0.228l1.661 1.663c-0.932 0.463-2.052 0.276-2.787-0.459zM25.271 10.125c-0.109 0.109-0.229 0.208-0.359 0.291l-1.609-1.609 2.041-2.047c0.885 0.964 0.849 2.443-0.073 3.365zM25.14 8.068c-0.067 0.047-0.093 0.129-0.072 0.208 0.099 0.197 0.072 0.432-0.068 0.599-0.068 0.084-0.052 0.199 0.031 0.265 0.032 0.021 0.068 0.037 0.109 0.037 0.057 0 0.111-0.021 0.141-0.063 0.235-0.281 0.281-0.677 0.12-1.005-0.063-0.083-0.177-0.104-0.261-0.041z"></path> </g></svg>
                        </span>
                        <span class="text-black">Postman</span>
                    </div>
                </div>
            </div>
            
        </div>
    </div>

    

</template>
