<div class="page-head">
    <i>16/04/2024 ~ #ray-tracing ~ #voxels ~ #grid-traversal</i>
    <h1><img src="assets/images/mushroom.png" title="RAM Stick"> Amanatides and Woo</h1>
    <p>Traversing through <span class="yellow">grids</span> quickly</p>
</div>

## Introduction

<div class="h-group">
<figure title="Figure A: Ray step (signs)">
    <svg class="fig" width="256" viewBox="0 0 194.5 194.5">
        <pattern id="grid32" width="32" height="32" patternUnits="userSpaceOnUse">
            <line x1="0" y1="0" x2="0" y2="32" stroke="#342c28" stroke-width="4" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="#342c28" stroke-width="4" />
        </pattern>
        <pattern id="grid64" width="64" height="64" patternUnits="userSpaceOnUse">
            <rect width="64" height="64" fill="url(#grid32)" />
            <line x1="0" y1="0" x2="0" y2="64" stroke="#3d3a34" stroke-width="5" />
            <line x1="0" y1="0" x2="64" y2="0" stroke="#3d3a34" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="194.5" height="194.5" rx="3px" ry="3px" fill="url(#grid64)" />
        <g>
            <text x="3.1620638" y="23.12842" fill="#3d3a34" font-size="20.05px" letter-spacing="-4px" stroke-dasharray="13.3663, 13.3663" stroke-linecap="round" stroke-linejoin="round" stroke-width="3.3416" xml:space="preserve"><tspan x="3.1620638" y="23.12842" fill="#3d3a34" font-family="'JetBrains Mono'" font-weight="800" letter-spacing="-4px" stroke-width="3.3416">0,0</tspan></text>
            <g fill="#ffffff" font-size="20.05px" stroke-dasharray="13.3663, 13.3663" stroke-linecap="round" stroke-linejoin="round" stroke-width="3.3416">
            <text x="108.10543" y="135.54257" letter-spacing="0px" xml:space="preserve"><tspan x="108.10543" y="135.54257" fill="#ffffff" font-family="'JetBrains Mono'" font-weight="800" letter-spacing="0px" stroke-width="3.3416">1</tspan></text>
            <text x="66.967758" y="104.56811" letter-spacing="0px" xml:space="preserve"><tspan x="66.967758" y="104.56811" fill="#ffffff" font-family="'JetBrains Mono'" font-weight="800" letter-spacing="0px" stroke-width="3.3416">-1</tspan></text>
            <text x="109.36742" y="84.158424" xml:space="preserve"><tspan x="109.36742" y="84.158424" fill="#ffffff" font-family="'JetBrains Mono'" font-weight="800" stroke-width="3.3416">step</tspan></text>
            </g>
            <g stroke-dashoffset="21.9" stroke-linecap="round" stroke-linejoin="round" stroke-width="3">
            <path d="m97.25 114.03v-32.034" fill="#fff" fill-opacity=".4" stroke="#fff"/>
            <path d="m129.28 109.62v8.824l6.2052-4.412z" fill="#fff" stroke="#fff"/>
            <path d="m187.15 77.817 3.0598 8.2766 4.2904-6.29z" fill="#fed436" stroke="#fed436"/>
            </g>
            <g fill="#fff">
            <g stroke-dashoffset="21.9" stroke-linecap="round" stroke-linejoin="round" stroke-width="3">
            <path d="m92.838 81.998h8.8241l-4.412-6.2052z" stroke="#fff"/>
            <path d="m0 148.26 194.5-68.456" fill-opacity=".4" stroke="#fed436"/>
            <path d="m97.25 114.03h32.034" fill-opacity=".4" stroke="#fff"/>
            </g>
            <circle cx="97.25" cy="114.03" r="6"/>
            </g>
        </g>
    </svg>
</figure>
<figure title="Figure B: Ray delta (reciprocal dir)">
    <svg class="fig" width="256" viewBox="0 0 194.5 194.5">
        <pattern id="grid32" width="32" height="32" patternUnits="userSpaceOnUse">
            <line x1="0" y1="0" x2="0" y2="32" stroke="#342c28" stroke-width="4" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="#342c28" stroke-width="4" />
        </pattern>
        <pattern id="grid64" width="64" height="64" patternUnits="userSpaceOnUse">
            <rect width="64" height="64" fill="url(#grid32)" />
            <line x1="0" y1="0" x2="0" y2="64" stroke="#3d3a34" stroke-width="5" />
            <line x1="0" y1="0" x2="64" y2="0" stroke="#3d3a34" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="194.5" height="194.5" rx="3px" ry="3px" fill="url(#grid64)" />
        <g>
            <g stroke-linecap="round" stroke-linejoin="round">
            <g>
            <g font-size="20.05px" stroke-dasharray="13.3663, 13.3663" stroke-width="3.3416">
                <text x="3.1620638" y="23.12842" fill="#3d3a34" letter-spacing="-4px" xml:space="preserve"><tspan x="3.1620638" y="23.12842" fill="#3d3a34" font-family="'JetBrains Mono'" font-weight="800" letter-spacing="-4px" stroke-width="3.3416">0,0</tspan></text>
                <text transform="rotate(-19.392)" x="24.338623" y="116.30575" fill="#ffffff" xml:space="preserve"><tspan x="24.338623" y="116.30575" fill="#ffffff" font-family="'JetBrains Mono'" font-weight="800" stroke-width="3.3416">delta<tspan baseline-shift="sub" font-size="65%">y</tspan></tspan></text>
                <text transform="rotate(-19.392)" x="4.6828022" y="174.56879" fill="#ffffff" xml:space="preserve"><tspan x="4.6828022" y="174.56879" fill="#ffffff" font-family="'JetBrains Mono'" font-weight="800" stroke-width="3.3416">delta<tspan baseline-shift="sub" font-size="65%">x</tspan></tspan></text>
            </g>
            <path d="m187.15 77.817 3.0598 8.2766 4.2904-6.29z" fill="#7e4d05" stroke="#7e4d05" stroke-dashoffset="21.9" stroke-width="3"/>
            </g>
            <g fill="none" stroke-dashoffset="21.9" stroke-width="3">
            <path d="m144.69 97.014-4.0816-11.643-90.342 31.87 4.0816 11.643" stroke="#fff"/>
            <path d="m65.25 125.62 4.0816 11.643 31.974-11.286-4.1065-11.634" stroke="#fff"/>
            <path d="m0 148.26 194.5-68.456" stroke="#7e4d05"/>
            </g>
            <g fill="#7b3333" stroke="#fed436" stroke-dashoffset="21.9" stroke-width="3">
            <path d="m17.25 97.25v32"/>
            <path d="m11.879 97.25h10.742"/>
            <path d="m65.25 171.88v10.742"/>
            <path d="m97.25 171.88v10.742"/>
            <path d="m11.879 129.25h10.742"/>
            <path d="m65.25 177.25h32"/>
            </g>
            </g>
        </g>
    </svg>
</figure>
</div>

Some text...

<div class="h-group">
<figure title="Figure C: Finding entry cell by truncating">
    <svg class="fig" width="256" viewBox="0 0 194.5 194.5">
        <pattern id="grid32" width="32" height="32" patternUnits="userSpaceOnUse">
            <line x1="0" y1="0" x2="0" y2="32" stroke="#342c28" stroke-width="4" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="#342c28" stroke-width="4" />
        </pattern>
        <pattern id="grid64" width="64" height="64" patternUnits="userSpaceOnUse">
            <rect width="64" height="64" fill="url(#grid32)" />
            <line x1="0" y1="0" x2="0" y2="64" stroke="#3d3a34" stroke-width="5" />
            <line x1="0" y1="0" x2="64" y2="0" stroke="#3d3a34" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="194.5" height="194.5" rx="3px" ry="3px" fill="url(#grid64)" />
        <g>
            <path d="m64 81.304 130.5 33.599" fill="#fff" stroke="#7e4d05" stroke-linecap="round" stroke-linejoin="round" stroke-width="3"/>
            <path d="m65.249 194.5 7e-4 -129.25 129.25 0.16817" fill="none" stroke="#fff" stroke-opacity=".53333" stroke-dasharray="12, 12" stroke-dashoffset="21.9" stroke-linecap="round" stroke-linejoin="round" stroke-width="3"/>
            <g stroke-linecap="round" stroke-linejoin="round">
            <text x="93.092438" y="134.0918" fill="#ffffff" fill-opacity=".53333" font-size="20.05px" stroke-dasharray="13.3663, 13.3663" stroke-width="3.3416" xml:space="preserve"><tspan x="93.092438" y="134.0918" fill="#ffffff" fill-opacity=".53333" font-family="'JetBrains Mono'" font-weight="800" stroke-width="3.3416">object</tspan></text>
            <text x="4.1620638" y="24.12842" fill="#3d3a34" font-size="20.05px" stroke-dasharray="13.3663, 13.3663" stroke-width="3.3416" letter-spacing="-4px" xml:space="preserve"><tspan x="3.1620638" y="24.12842" fill="#3d3a34" font-family="'JetBrains Mono'" font-weight="800" letter-spacing="-4px" stroke-width="3.3416">0,0</tspan></text>
            <text x="49.56469" y="46.115707" fill="#fed436" fill-opacity="0" font-size="20.05px" stroke-dasharray="13.3663, 13.3663" stroke-width="3.3416" xml:space="preserve"><tspan x="49.56469" y="46.115707" fill="#fed436" font-family="'JetBrains Mono'" font-weight="800" stroke-width="3.3416">entry cell</tspan>
                <animate
                    attributeName="fill-opacity"
                    values="0;0;0;0;1;1;1;0;0;0;0"
                    dur="10s"
                    repeatCount="indefinite" />
            </text>
            <path d="m0 64 64 17.304" fill="#fff" stroke="#fed436" stroke-width="3"/>
            </g>
            <rect x="65.25" y="65.25" width="32" height="32" fill="none" stroke="#fed436" stroke-dashoffset="200" stroke-dasharray="200" stroke-linecap="round" stroke-linejoin="round" stroke-width="3">
                <animate
                    attributeName="stroke-dashoffset"
                    values="200;200;200;200;0;0;0;200;200;200;200"
                    dur="10s"
                    repeatCount="indefinite" />
            </rect>
            <circle cx="65.5" cy="65.257" r="6" fill="#fff">
                <animate
                    attributeName="cy"
                    values="81.686;81.686;65.257;65.257;65.257;65.257;81.686;81.686"
                    dur="10s"
                    repeatCount="indefinite" />
            </circle>
            <path d="m189.52 109.14-2.0996 8.5706 7.0768-2.8089z" fill="#7e4d05" stroke="#7e4d05" stroke-dashoffset="21.9" stroke-linecap="round" stroke-linejoin="round" stroke-width="3"/>
        </g>
    </svg>
</figure>
<figure title="Figure D: Time at next cell boundary (tmax)">
    <svg class="fig" width="256" viewBox="0 0 194.5 194.5">
        <pattern id="grid32" width="32" height="32" patternUnits="userSpaceOnUse">
            <line x1="0" y1="0" x2="0" y2="32" stroke="#342c28" stroke-width="4" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="#342c28" stroke-width="4" />
        </pattern>
        <pattern id="grid64" width="64" height="64" patternUnits="userSpaceOnUse">
            <rect width="64" height="64" fill="url(#grid32)" />
            <line x1="0" y1="0" x2="0" y2="64" stroke="#3d3a34" stroke-width="5" />
            <line x1="0" y1="0" x2="64" y2="0" stroke="#3d3a34" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="194.5" height="194.5" rx="3px" ry="3px" fill="url(#grid64)" />
        <g>
            <g stroke-linecap="round" stroke-linejoin="round">
            <rect x="65.25" y="65.25" width="32" height="32" fill="none" stroke="#7b3333" stroke-dashoffset="21.9" stroke-width="3"/>
            <text x="3.1620638" y="23.12842" fill="#3d3a34" font-size="20.05px" letter-spacing="-4px" stroke-dasharray="13.3663, 13.3663" stroke-width="3.3416" xml:space="preserve"><tspan x="3.1620638" y="23.12842" fill="#3d3a34" font-family="'JetBrains Mono'" font-weight="800" letter-spacing="-4px" stroke-width="3.3416">0,0</tspan></text>
            <path d="m189.52 109.14-2.0996 8.5706 7.0768-2.8089z" fill="#7e4d05" stroke="#7e4d05" stroke-dashoffset="21.9" stroke-width="3"/>
            <rect x="61.5" y="11.5" width="10.971" height="10.971" fill="none" stroke="#7b3333" stroke-dashoffset="21.9" stroke-width="3"/>
            <text transform="rotate(-.16513)" x="80.952148" y="20.707891" fill="#7b3333" font-size="10.716px" stroke-dasharray="13.3663, 13.3663" stroke-width="3.3416" xml:space="preserve"><tspan x="80.952148" y="20.707891" fill="#7b3333" font-family="'JetBrains Mono'" font-size="10.716px" font-weight="800" stroke-width="3.3416">current position</tspan></text>
            </g>
            <g fill="#fff">
            <path d="m64 81.304 130.5 33.599" stroke="#7e4d05" stroke-linecap="round" stroke-linejoin="round" stroke-width="3"/>
            <path d="m0 64 64 17.304" stroke="#fed436" stroke-linecap="round" stroke-linejoin="round" stroke-width="3"/>
            <circle cx="65.5" cy="81.25" r="6"/>
            </g>
            <text transform="rotate(14.698)" x="119.03473" y="40.408497" fill="#ffffff" font-size="20.05px" stroke-dasharray="13.3663, 13.3663" stroke-linecap="round" stroke-linejoin="round" stroke-width="3.3416" xml:space="preserve"><tspan x="119.03473" y="40.408497" fill="#ffffff" font-family="'JetBrains Mono'" font-weight="800" stroke-width="3.3416">tmax<tspan baseline-shift="sub" font-size="65%">y</tspan></tspan></text>
            <text transform="rotate(14.698)" x="87.346458" y="95.128494" fill="#ffffff" font-size="20.05px" stroke-dasharray="13.3663, 13.3663" stroke-linecap="round" stroke-linejoin="round" stroke-width="3.3416" xml:space="preserve"><tspan x="87.346458" y="95.128494" fill="#ffffff" font-family="'JetBrains Mono'" font-weight="800" stroke-width="3.3416">tmax<tspan baseline-shift="sub" font-size="65%">x</tspan></tspan></text>
            <g fill="none" stroke="#f9f9f9" stroke-dashoffset="21.9" stroke-linecap="round" stroke-linejoin="round" stroke-width="3">
            <path d="m135.58 86.063-12.06-3.1054"/>
            <path d="m99.981 104.31-12.06-3.1054"/>
            <path d="m129.55 84.511-3.3008 12.821"/>
            <path d="m93.95 102.75 3.3171-12.884"/>
            </g>
        </g>
    </svg>
</figure>
</div>
