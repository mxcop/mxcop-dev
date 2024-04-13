<div class="page-head">
    <i>13/04/2024 ~ #ray-tracing ~ #voxels ~ #optimization</i>
    <h1>Coherent Voxel Traversal</h1>
    <p>Defeating the <span class="yellow">memory bottleneck</span></p>
</div>

Hello, world!

<figure>
  <svg class="fig-scale" width="550" viewBox="-80 0 550 400">
    <g visibility="hidden">
      <foreignObject x="160" y="185" width="80" height="25">
        $ x \gt x_{min} $
      </foreignObject>
      <set id="reset" attributeName="visibility" to="hidden" begin="0s; edges.end" dur="4s" />
      <set id="xmin" attributeName="visibility" to="visible" begin="reset.end" dur="2s" />
      <set attributeName="visibility" to="hidden" begin="xmin.end" end="edges.begin" />
      <set id="edges" attributeName="visibility" to="hidden" begin="ymax.end" dur="4s" />
    </g>
    <g visibility="hidden">
      <foreignObject x="160" y="185" width="80" height="25">
        $ x \lt x_{max} $
      </foreignObject>
      <set id="xmax" attributeName="visibility" to="visible" begin="xmin.end" dur="2s" />
      <set attributeName="visibility" to="hidden" begin="xmax.end" end="xmax.begin" />
    </g>
    <g visibility="hidden">
      <foreignObject x="160" y="185" width="80" height="25">
        $ y \gt y_{min} $
      </foreignObject>
      <set id="ymin" attributeName="visibility" to="visible" begin="xmax.end" dur="2s" />
      <set attributeName="visibility" to="hidden" begin="ymin.end" end="ymin.begin" />
    </g>
    <g visibility="hidden">
      <foreignObject x="160" y="185" width="80" height="25">
        $ y \lt y_{max} $
      </foreignObject>
      <set id="ymax" attributeName="visibility" to="visible" begin="ymin.end" dur="2s" />
      <set attributeName="visibility" to="hidden" begin="ymax.end" end="ymax.begin" />
    </g>
    <pattern id="hatch" patternUnits="userSpaceOnUse" width="8" height="8" patternTransform="rotate(45)">
        <line x1="0" y1="0" x2="0" y2="8" stroke="var(--icons)" stroke-width="1" />
      </pattern>
    <rect x="-80" y="0" width="550" height="400" fill="url(#hatch)">
      <animate attributeName="x" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="width" to="390" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="width" to="240" begin="xmax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y" to="80" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="height" to="320" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="height" to="240" begin="ymax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y" to="0" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="width" to="550" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="height" to="400" begin="reset.begin" dur="1s" fill="freeze" />
    </rect>
    <line x1="-81" y1="0" x2="-81" y2="400" stroke="var(--fg)">
      <animate attributeName="x1" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y1" to="80" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="320" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-81" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="0" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="-81" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="400" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="471" y1="0" x2="471" y2="400" stroke="var(--fg)">
      <animate attributeName="x1" to="320" begin="xmax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="xmax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y1" to="80" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="320" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="471" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="0" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="471" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="400" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="-1" x2="470" y2="-1" stroke="var(--fg)">
      <animate attributeName="y1" to="80" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="80" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="80" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="-1" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="470" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="-1" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="401" x2="470" y2="401" stroke="var(--fg)">
      <animate attributeName="y1" to="320" begin="ymax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="320" begin="ymax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="80" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="401" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="470" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="401" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
  </svg>
</figure>

<figure>
  <svg class="fig-scale" width="550" viewBox="-80 0 550 400">
    <line x1="-81" y1="0" x2="-81" y2="400" stroke="var(--fg)">
      <animate attributeName="x1" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y1" to="80" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="320" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-81" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="0" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="-81" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="400" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="471" y1="0" x2="471" y2="400" stroke="var(--fg)">
      <animate attributeName="x1" to="320" begin="xmax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="xmax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y1" to="80" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="320" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="471" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="0" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="471" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="400" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="-1" x2="470" y2="-1" stroke="var(--fg)">
      <animate attributeName="y1" to="80" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="80" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="80" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="-1" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="470" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="-1" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="401" x2="470" y2="401" stroke="var(--fg)">
      <animate attributeName="y1" to="320" begin="ymax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="320" begin="ymax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="80" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="401" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="470" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="401" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="200" x2="120" y2="0" stroke="var(--icons)" stroke-dasharray="1" />
    <line x1="-80" y1="200" x2="120" y2="0" stroke="var(--fg)" stroke-width="2">
      <animate attributeName="x1" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y1" to="40" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="40" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="80" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="stroke" to="red" begin="ymin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="200" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="120" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="0" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="stroke" to="var(--fg)" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="200" x2="386.666666" y2="0" stroke="var(--icons)" stroke-dasharray="1" />
    <line x1="-80" y1="200" x2="386.666666" y2="0" stroke="var(--fg)" stroke-width="2">
      <animate attributeName="x1" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y1" to="131.428571" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="xmax.begin+0.277777s" dur="0.222222s" fill="freeze" />
      <animate attributeName="y2" to="28.571428" begin="xmax.begin+0.277777s" dur="0.222222s" fill="freeze" />
      <animate attributeName="x2" to="200" begin="ymin.begin+0.17857142s" dur="0.32142857s" fill="freeze" />
      <animate attributeName="y2" to="80" begin="ymin.begin+0.17857142s" dur="0.32142857s" fill="freeze" />
      <animate attributeName="stroke" to="green" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="200" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="386.666666" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="0" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="stroke" to="var(--fg)" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="200" x2="470" y2="200" stroke="var(--icons)" stroke-dasharray="1" />
    <line x1="-80" y1="200" x2="470" y2="200" stroke="var(--fg)" stroke-width="2">
      <animate attributeName="x1" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="xmax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="stroke" to="green" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="470" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="stroke" to="var(--fg)" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="200" x2="386.666666" y2="400" stroke="var(--icons)" stroke-dasharray="1" />
    <line x1="-80" y1="200" x2="386.666666" y2="400" stroke="var(--fg)" stroke-width="2">
      <animate attributeName="x1" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y1" to="268.571428" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="320" begin="xmax.begin+0.277777s" dur="0.222222s" fill="freeze" />
      <animate attributeName="y2" to="371.428571" begin="xmax.begin+0.277777s" dur="0.222222s" fill="freeze" />
      <animate attributeName="x2" to="200" begin="ymax.begin+0.17857142s" dur="0.32142857s" fill="freeze" />
      <animate attributeName="y2" to="320" begin="ymax.begin+0.17857142s" dur="0.32142857s" fill="freeze" />
      <animate attributeName="stroke" to="green" begin="edges.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="200" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="386.666666" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="400" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="stroke" to="var(--fg)" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <line x1="-80" y1="200" x2="120" y2="400" stroke="var(--icons)" stroke-dasharray="1" />
    <line x1="-80" y1="200" x2="120" y2="400" stroke="var(--fg)" stroke-width="2">
      <animate attributeName="x1" to="80" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y1" to="360" begin="xmin.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x2" to="40" begin="ymax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="y2" to="320" begin="ymax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="stroke" to="red" begin="ymax.begin" dur="0.5s" fill="freeze" />
      <animate attributeName="x1" to="-80" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y1" to="200" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="x2" to="120" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="y2" to="400" begin="reset.begin" dur="1s" fill="freeze" />
      <animate attributeName="stroke" to="var(--fg)" begin="reset.begin" dur="1s" fill="freeze" />
    </line>
    <rect x="80" y="80" width="240" height="240" stroke="var(--fg)" fill="url(#hatch)" />
  </svg>
</figure>
