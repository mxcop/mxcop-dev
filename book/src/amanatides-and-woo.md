<div class="page-head">
    <i>16/04/2024 ~ #ray-tracing ~ #voxels ~ #grid-traversal</i>
    <h1><img src="assets/images/mushroom.png" title="RAM Stick"> Amanatides and Woo</h1>
    <p>Traversing through <span class="yellow">grids</span> quickly</p>
</div>

## Introduction

In this series of articles we're going to be diving into grid / voxel traversal and how to make it faster!<br>
The algorithm we're talking about is [*Amanatides and Woo "A Fast Voxel Traversal Algorithm for Ray Tracing"*](https://www.researchgate.net/publication/2611491_A_Fast_Voxel_Traversal_Algorithm_for_Ray_Tracing)

We will not go into great detail here as to how it exactly works.<br>
[This page](https://github.com/cgyurgyik/fast-voxel-traversal-algorithm/blob/master/overview/FastVoxelTraversalOverview.md) has a good overview of the algorithm and how it works.<br>
Sometimes we may refer to the traversal algorithm(s) as "DDA" *Digital Differential Analyzer*.

*The basic idea is:*<br>
We calculate the maximum distance we can traverse before we pass a grid boundary on each axis `tmax`.<br>
The smallest axis of `tmax` is the axis we should step on next.

<figure title="Figure A: Visualization of tmax.">
    <svg class="fig" width="256" viewBox="0 0 130 130">
        <pattern id="grid" patternUnits="userSpaceOnUse" width="32" height="32">
            <line x1="0" y1="0" x2="0" y2="32" stroke="var(--fig-w20)" stroke-width="5" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="var(--fig-w20)" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="130" height="130" fill="url(#grid)" />
        <g>
            <line x1="36" y1="0" x2="82" y2="128" stroke="var(--fig-y50)" stroke-width="2" stroke-linecap="round" stroke-dasharray="6" />
            <rect x="33" y="65" width="32" height="32" fill="none" stroke="var(--fig-error-50)" stroke-width="2.5" stroke-linecap="round" />
            <foreignObject x="36" y="67" width="60" height="25" color="var(--fig-error-50)">
                $ pos $
            </foreignObject>
        </g>
        <g visibility="hidden">
            <foreignObject x="6" y="64" width="60" height="25" color="var(--fig-y90)">
                $ tmax_x $
            </foreignObject>
            <line x1="59" y1="64" x2="65" y2="80" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" />
            <line x1="65" y1="70" x2="65" y2="86" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" />
            <set id="tmaxX" attributeName="visibility" to="visible" begin="0;tmaxY.end" dur="4.0s" />
        </g>
        <g visibility="hidden">
            <foreignObject x="72" y="68" width="60" height="25" color="var(--fig-y90)">
                $ tmax_y $
            </foreignObject>
            <line x1="59" y1="64" x2="70.5" y2="96" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" />
            <line x1="66.5" y1="97" x2="74.5" y2="97" stroke="var(--fig-y90)" stroke-width="2" stroke-linecap="round" />
            <set id="tmaxY" attributeName="visibility" to="visible" begin="tmaxX.end" dur="4.0s" />
        </g>
        <g>
            <line x1="36" y1="0" x2="58" y2="62" stroke="var(--fig-error)" stroke-width="2" stroke-linecap="round" marker-end="url(#arrow-error)" />
        </g>
    </svg>
</figure>

As we can see in *Figure A*, with `pos` as the current cell,<br>
the next step should indeed be on the axis where `tmax` is the smallest.<br>
Then, we update `tmax` by adding the reciprocal ray direction to the smallest axis<br>
and move `pos` by adding the ray direction sign to the smallest axis of `pos`.

## The problem

During my journey ray tracing voxels on the CPU, the first thing that became apparent to me was<br>
that all my traversal algorithms were bottlenecked by poor cache utilization.

Traversing grids is not great for our caches, often causing many many <span class="yellow">cache misses</span>.

For example, we often store our grids in a 2D/3D array in memory.<br>
The resulting layout would look something like *Figure B*.
<div class="h-group">
<figure title="Figure B: Typical grid order in memory.">
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
            <path
       style="fill:none;stroke:#ffffff;stroke-width:2;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:none"
       d="M 0,64 64,0 128,64 192,0 256,64 128,192 Z"
       id="path1" />
        </g>
    </svg>
</figure>
<figure title="Figure C: The good and the horrible...">
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

*Figure C* shows two rays, **green** with a good access pattern, and **red** with a terrible one.<br>
The **red** ray has to jump all over the 2D/3D array in memory, causing many cache misses.

*This effect gets exaggerated as the grid gets larger.*

## Cache misses?

Our modern CPUs have multiple levels of cache memory *(L1, L2, L3)*<br>
I will not go into great detail about it here.<br>

*However, what we need to know is :*<br>
Whenever we fetch some memory, our CPUs will not only fetch exactly what we requested.<br>
Instead it will fetch a block of memory, called a <span class="yellow">cache line</span> containing our requested memory.<br>
Which will be placed into our caches, in case we may need anything from that cache line later.<br>

For most modern CPUs a cache line is `64 bytes` in length and for GPUs its `128 bytes`.<br>
Important to note, cache lines are always aligned to their size *(64 in most cases)*<br>

As an example, lets say each cell in our 8x8 grid is `8 bytes` in size.<br>
When we read a cell from memory, the cache line would contain `8 cells` worth of data.

<figure title="Figure D: CPU Loading a cache line.">
    <svg class="fig" width="256" viewBox="0 0 258 258">
        <pattern id="grid" patternUnits="userSpaceOnUse" width="32" height="32">
            <line x1="0" y1="0" x2="0" y2="32" stroke="var(--fig-w20)" stroke-width="5" />
            <line x1="0" y1="0" x2="32" y2="0" stroke="var(--fig-w20)" stroke-width="5" />
        </pattern>
        <rect x="0" y="0" width="258" height="258" fill="url(#grid)" />
        <g>
            <rect x="161" y="65" width="32" height="32" fill="none" stroke="var(--fig-y50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="193" y="65" width="32" height="32" fill="none" stroke="var(--fig-y50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="225" y="65" width="32" height="32" fill="none" stroke="var(--fig-y50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="129" y="97" width="32" height="32" fill="none" stroke="var(--fig-y50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="97" y="97" width="32" height="32" fill="none" stroke="var(--fig-y50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="33" y="97" width="32" height="32" fill="none" stroke="var(--fig-y50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="1" y="97" width="32" height="32" fill="none" stroke="var(--fig-y50)" stroke-width="2.5" stroke-linecap="round" />
            <rect x="65" y="97" width="32" height="32" fill="none" stroke="var(--fig-y90)" stroke-width="2.5" stroke-linecap="round" />
        </g>
    </svg>
</figure>

*Figure D* shows what a cache line *may* look like in our 8x8 grid.<br>
*(it depends on how our grid is aligned in memory)*

This is nice if our next step in the grid is to the right.<br>
Because those cells are already in our cache now, so we don't need to go back to RAM. *(RAM is slow)*

However, in a lot of cases we will move up or down instead.<br>
Because this memory isn't in our cache, we would cause what is called a <span class="yellow">cache miss</span>.<br>
In most modern programs this is not a problem at all.<br>
However, it is when we are shooting <span class="yellow">millions of rays</span> per second.

## Space filling curves

We can change the layout of the grid in memory.<br>
Space filling curves are a great option that let us improve the <span class="yellow">spatial locality</span> of our grids.<br>
*Basically*, cells close to each other in space, are also close to each other in memory.

<div class="h-group">
<figure title="Figure E: Morton curve.">
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
<figure title="Figure F: Hilbert curve.">
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

*Figure E & F* show two beautiful space filling curves. *(these also extend into 3D)*<br>
When we fetch a cell in these new grids, the cache line will contain much more useful additional cells.<br>

The computation required to encode a 2D/3D coordinate onto a Hilbert curve is rather large.<br>
However it turns out that the Morton encoding is *cheap enough* to make its use viable.

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

To encode a 2D/3D coordinate onto a Morton curve, we can use the `pdep` instruction on x86.<br>
*Snippet A* shows how we can do this in C++.<br>
This is nice and fast, because it only requires a few instructions.

*So? What are your gains you may be wondering.*<br>
For me it was a bit inconsistent, however it did consistently reduce my frametimes, albeit <span class="yellow">not by much</span>.<br>
I recon this is because of the extra cost of encoding the coordinates each step.

## Limitations

Space filling curves tend to have some constraints.<br>
For example in the case of our Morton curve, it only works in <span class="yellow">powers of 2</span>.<br>
So our grid has to be `2x2, 4x4, 8x8, 16x16, 32x32, 64x64, ...`. *(the same goes for 3D)*

Furthermore, with a Morton curve our grid **MUST** be a <span class="yellow">perfect cube</span>.<br>
As in, all sides must be of the same length.

## Conclusion

So, We've looked at the main bottleneck of DDA and a way to make it slightly faster.<br>

*What's next?*<br>
There are plenty of other ways to make our grid traversal faster.<br>
We will be going into 2 algorithms in the following chapters that do just that.<br>
