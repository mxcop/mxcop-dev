<div class="page-head">
    <i>13/04/2024 ~ #ray-tracing ~ #voxels ~ #optimization</i>
    <h1><img src="../assets/images/ram-stick.png" title="RAM Stick"> Coherent Voxel Traversal</h1>
    <p>Surpassing the <span class="yellow">memory bottleneck</span></p>
</div>

## The problem

During my journey ray tracing voxels on the CPU, the first thing that became apparent to me was;<br>
all my traversal algorithms were bottlenecked by poor cache utilization.

Traversing grids is not great for our caches, for we cannot assure a <span class="yellow">linear access pattern</span>.

For example, we often store our voxels in a 2D/3D array in memory.<br>
The resulting layout would look something like *Figure A*.
<div class="h-group">
<figure title="Figure A: Typical grid order in memory.">
    <svg class="fig" width="256" viewBox="0 0 258 258">
        <pattern id="grid" patternUnits="userSpaceOnUse" width="32" height="32">
            <line x1="0" y1="0" x2="0" y2="32" stroke="var(--fig-w20)" stroke-width="5" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="var(--fig-w20)" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="258" height="258" fill="url(#grid)" />
        <g>
            <defs>
                <marker id="arrow" viewBox="0 -5 10 10" refX="5" refY="0" markerWidth="4" markerHeight="4" orient="auto">
                    <path stroke="var(--fig-y90)" fill="var(--fig-y90)" d="M0,-5L10,0L0,5" />
                </marker>
            </defs>
            <line x1="241" y1="17" x2="17" y2="49" stroke="var(--fig-y50)" stroke-width="2" stroke-linecap="round" />
            <line x1="241" y1="49" x2="17" y2="81" stroke="var(--fig-y50)" stroke-width="2" stroke-linecap="round" />
            <line x1="241" y1="81" x2="17" y2="113" stroke="var(--fig-y50)" stroke-width="2" stroke-linecap="round" />
            <line x1="241" y1="113" x2="17" y2="145" stroke="var(--fig-y50)" stroke-width="2" stroke-linecap="round" />
            <line x1="241" y1="145" x2="17" y2="177" stroke="var(--fig-y50)" stroke-width="2" stroke-linecap="round" />
            <line x1="241" y1="177" x2="17" y2="209" stroke="var(--fig-y50)" stroke-width="2" stroke-linecap="round" />
            <line x1="241" y1="209" x2="17" y2="241" stroke="var(--fig-y50)" stroke-width="2" stroke-linecap="round" />
            <line x1="17" y1="17" x2="241" y2="17" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow)" />
            <line x1="17" y1="49" x2="241" y2="49" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow)" />
            <line x1="17" y1="81" x2="241" y2="81" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow)" />
            <line x1="17" y1="113" x2="241" y2="113" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow)" />
            <line x1="17" y1="145" x2="241" y2="145" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow)" />
            <line x1="17" y1="177" x2="241" y2="177" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow)" />
            <line x1="17" y1="209" x2="241" y2="209" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow)" />
            <line x1="17" y1="241" x2="241" y2="241" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow)" />
        </g>
    </svg>
</figure>
<figure title="Figure B: The good and the horrible...">
    <svg class="fig" width="256" viewBox="0 0 258 258">
        <pattern id="grid" patternUnits="userSpaceOnUse" width="32" height="32">
            <line x1="0" y1="0" x2="0" y2="32" stroke="var(--fig-w20)" stroke-width="5" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="var(--fig-w20)" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="258" height="258" fill="url(#grid)" />
        <g opacity="0">
            <defs>
                <marker id="arrow-success" viewBox="0 -5 10 10" refX="5" refY="0" markerWidth="4" markerHeight="4" orient="auto">
                    <path stroke="var(--fig-success)" fill="var(--fig-success)" d="M0,-5L10,0L0,5" />
                </marker>
            </defs>
            <rect x="1" y="97" width="32" height="32" fill="none" stroke="var(--fig-success-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="33" y="97" width="32" height="32" fill="none" stroke="var(--fig-success-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="65" y="97" width="32" height="32" fill="none" stroke="var(--fig-success-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="97" y="97" width="32" height="32" fill="none" stroke="var(--fig-success-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="129" y="97" width="32" height="32" fill="none" stroke="var(--fig-success-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="161" y="97" width="32" height="32" fill="none" stroke="var(--fig-success-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="193" y="97" width="32" height="32" fill="none" stroke="var(--fig-success-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="225" y="97" width="32" height="32" fill="none" stroke="var(--fig-success-50)" stroke-width="2.5" stroke-linecap="round" />
            <line x1="0" y1="109" x2="254" y2="117" stroke="var(--fig-success)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow-success)" />
            <animate id="goodRayEntry" attributeName="opacity" to="5" begin="0;badRayExit.end" dur="5.0s" fill="freeze" />
            <animate id="goodRayExit" attributeName="opacity" to="0" begin="goodRayEntry.end" dur="5.0s" fill="freeze" />
        </g>
        <g opacity="0">
            <defs>
                <marker id="arrow-error" viewBox="0 -5 10 10" refX="5" refY="0" markerWidth="4" markerHeight="4" orient="auto">
                    <path stroke="var(--fig-error)" fill="var(--fig-error)" d="M0,-5L10,0L0,5" />
                </marker>
            </defs>
            <rect x="65" y="1" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="65" y="33" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="97" y="33" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="97" y="65" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="97" y="97" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="129" y="97" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="129" y="129" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="129" y="161" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="129" y="193" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="161" y="193" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="161" y="225" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <line x1="74" y1="0" x2="184" y2="254" stroke="var(--fig-error)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow-error)" />
            <animate id="badRayEntry" attributeName="opacity" to="5" begin="goodRayExit.end" dur="5.0s" fill="freeze" />
            <animate id="badRayExit" attributeName="opacity" to="0" begin="badRayEntry.end" dur="5.0s" fill="freeze" />
        </g>
    </svg>
</figure>
</div>

*Figure B* shows two rays, **green** with a good access pattern, and **red** with a terrible one.<br>
The **red** ray has to jump all over the 2D/3D array in memory, causing many cache misses.


## Space filling curves

One of my first attempts to fix this issue was using space filling curves.<br>
These beautiful curves let us improve the <span class="yellow">spatial locality</span> of our grids.<br>
This means cells close to each other in space, are also close to each other in memory.

<div class="h-group">
<figure title="Figure C: Morton curve.">
    <svg class="fig" width="256" viewBox="0 0 258 258">
        <pattern id="grid" patternUnits="userSpaceOnUse" width="32" height="32">
            <line x1="0" y1="0" x2="0" y2="32" stroke="var(--fig-w20)" stroke-width="5" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="var(--fig-w20)" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="258" height="258" fill="url(#grid)" />
        <svg width="256" viewBox="20 299 222 222">
            <polyline fill="none" points="229.333,507.112 201.556,507.112 229.333,479.333   201.556,479.333 173.778,507.112 146,507.112 173.778,479.333 146,479.333 229.333,451.555 201.556,451.555 229.333,423.778   201.556,423.778 173.778,451.555 146,451.555 173.778,423.778 146,423.778 118.223,507.112 90.444,507.112 118.223,479.333   90.444,479.333 62.667,507.112 34.889,507.112 62.667,479.333 34.889,479.333 118.223,451.555 90.444,451.555 118.223,423.778   90.444,423.778 62.667,451.555 34.889,451.555 62.667,423.778 34.889,423.778 229.333,396 201.556,396 229.333,368.223   201.556,368.223 173.778,396 146,396 173.778,368.223 146,368.223 229.333,340.444 201.556,340.444 229.333,312.667   201.556,312.667 173.778,340.444 146,340.444 173.778,312.667 146,312.667 118.223,396 90.444,396 118.223,368.223 90.444,368.223   62.667,396 34.889,396 62.667,368.223 34.889,368.223 118.223,340.444 90.444,340.444 118.223,312.667 90.444,312.667   62.667,340.444 34.889,340.444 62.667,312.667 34.889,312.667  " stroke="var(--fig-y90)" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
    </svg>
</figure>
<figure title="Figure D: Hilbert curve.">
    <svg class="fig" width="256" viewBox="0 0 258 258">
        <pattern id="grid" patternUnits="userSpaceOnUse" width="32" height="32">
            <line x1="0" y1="0" x2="0" y2="32" stroke="var(--fig-w20)" stroke-width="5" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="var(--fig-w20)" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="258" height="258" fill="url(#grid)" />
        <svg width="256" viewBox="0 0 512 512">
            <path xmlns="http://www.w3.org/2000/svg" d="M32,32v64h64v-64h64h64v64h-64v64h64v64h-64h-64v-64h-64v64v64h64v64h-64v64v64h64v-64h64v64h64v-64v-64h-64v-64h64h64h64v64h-64v64v64h64v-64h64v64h64v-64v-64h-64v-64h64v-64v-64h-64v64h-64h-64v-64h64v-64h-64v-64h64h64v64h64v-64" fill="none" stroke="var(--fig-y90)" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
    </svg>
</figure>
</div>

*Figure C* shows a 2D Morton curve in our 8x8 example grid.<br>
When we place our voxels on such a curve, it is more likely the next voxel will already be in our caches.

<div>

```cpp
/* Encode 3D coordinate as a morton code. */
inline u64 morton_encode(const u64 x, const u64 y, const u64 z) {
    constexpr u64 BMI_3D_X_MASK = 0x9249249249249249;
    constexpr u64 BMI_3D_Y_MASK = 0x2492492492492492;
    constexpr u64 BMI_3D_Z_MASK = 0x4924924924924924;

    return _pdep_u64(x, BMI_3D_X_MASK) | _pdep_u64(y, BMI_3D_Y_MASK) | _pdep_u64(z, BMI_3D_Z_MASK);
}

/* Encode 2D coordinate as a morton code. */
inline u64 morton_encode(const u64 x, const u64 y) {
    constexpr u64 BMI_2D_X_MASK = 0x5555555555555555;
    constexpr u64 BMI_2D_Y_MASK = 0xAAAAAAAAAAAAAAAA;

    return _pdep_u64(x, BMI_2D_X_MASK) | _pdep_u64(y, BMI_2D_Y_MASK);
}
```
<sup>Snippet A.</sup>
</div>

To convert a regular 2D/3D index into a Morton code, we can use the `_pdep_u64` instruction on x86.<br>
*Snippet A* shows how we can do this in C++.

*Excellent! Problem solved, right?*<br>
Well... it turns out that encoding our regular indices to Morton codes in rather expensive.<br>
Even when using the `_pdep_u64` instruction on x86 the cost of encoding seems to almost/completely outweigh the gains.<br>
*(Your results may vary)*
