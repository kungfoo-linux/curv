===============
The Shape Atlas
===============
The Shape Atlas is a catalog of operations for constructing,
transforming and combining 2D and 3D geometric shapes.

This is a research project to identify, classify and implement
all of the artistically useful operations on signed distance fields
(the representation Curv uses for geometric shapes).

Each operation will be given a provisional API, and sometimes we'll explore multiple
implementations with different tradeoffs.

The ultimate goal is to boil all of this research down into a well
designed, consistent and powerful geometry API for Curv, which will be
included in a future Curv standard library.

2D and 3D Shapes:
  Every shape is marked as being 2-dimensional, 3-dimensional, or both.
  2D shapes are embedded in the XY plane.
  (The only standard shapes that are both are ``everything`` and ``nothing``.)

Infinite and Degenerate Shapes:
  A shape can be infinite. Many shape constructors accept ``inf`` as a dimension argument.

  A 2D shape with no area, or a 3D shape with no volume, is called degenerate.
  Examples are geometric points, line segments or curves, and in 3D, surfaces with 0 thickness.

  Points and curves are invisible in the preview window.
  Infinite and degenerate shapes are useful as intermediates for constructing
  shapes, even though you can't 3D print them or export them to some file formats.

Colour:
  Shapes have volumetric colour.
  A function assigns a colour to each point in the interior and on the boundary
  of every 2D and 3D shape. Primitive shapes are assigned a default yellowish colour,
  which you change using the ``colour`` function.
  Shape operations must specify how the colour of the result shape derives from the
  colour of the argument shapes.
  (Colour assignment and colour transformation operators will be researched elsewhere,
  in a Colour Atlas document.)

Signed Distance Fields:
  Shapes are represented internally as Signed Distance Fields (SDFs), see `<Theory.rst>`_.
  The issue here is that a given shape can be represented by many different SDFs, and it's not
  technically feasible to choose a single normalized SDF representation for all shapes.
  
  For each operation, we specify not just the shape that is constructed, but also the
  quality and structure of the resulting Signed Distance Field.
  In many cases, we specify multiple implementations of an operation with different SDF
  structures for the result. This usually represents an engineering tradeoff, where a
  better quality SDF costs more to compute.
  
  Some of the operations here operate not just on the shape of an argument,
  but also on its distance field, so that the shape of the result
  depends on the distance field of one or more arguments. These are "low level" operations,
  since they break the shape abstraction and expose the underlying implementation.
  
2D Shapes
=========
``circle d``
  Construct a circle of diameter ``d``, centred on the origin.
  Exact distance field.

``ellipse (dx, dy)``
  Construct an axis-aligned ellipse, centred on the origin,
  with width ``dx`` and height ``dy``.
  
  * ``ellipse_s``: scaled distance field, simple code, cheap to compute.
  * ``ellipse_e``: exact distance field, much more expensive to compute (TODO).

``square d``
  Construct an axis-aligned square of width ``d``, centred on the origin.
  
  * ``square_m``: mitred distance field, simple code, cheap to compute.
  * ``square_e``: exact distance field, more expensive.

``rect (dx, dy)``
  Construct an axis-aligned rectangle of width ``dx`` and height ``dy``,
  centred on the origin.
  
  * ``rect_m``: mitred distance field, simple code, cheap to compute.
  * ``rect_e``: exact distance field, more expensive.

``rect_at (lo, hi)``
  Construct an axis-aligned rectangle
  whose lower-left corner is ``lo``
  and whose upper-right corner is ``hi``.
  Unlike ``rect``, this function lets you construct
  half-infinite rectangles where, eg, the lower Y coordinate is
  finite but the upper Y coordinate is ``inf``.
  
  * ``rect_at_m``: mitred distance field
  * ``rect_at_e``: exact distance field (TODO)

``regular_polygon (n, d)``
  Construct a regular polygon, centred on the origin,
  with ``n`` sides, whose inscribed circle has diameter ``d``.
  Bottom edge is parallel to X axis.
  Cost: constant time and space, regardless of ``n``.
 
  * ``regular_polygon_m``: mitred distance field.
  * ``regular_polygon_e``: exact distance field (TODO).

..
  Example: ``regular_polygon(5,1)``

..
  |pentagon|

.. |pentagon| image:: images/pentagon.png

``convex_polygon vertices``
  Construct a convex polygon from a list of vertices in counter-clockwise order.
  The result is undefined if the vertex list doesn't specify a convex polygon.
  Cost: linear in ``len(vertices)``.
 
  * ``convex_polygon_m``: mitred distance field.
  * ``convex_polygon_e``: exact distance field (TODO).

``polygon vertices``
  TODO. (Use the Nef Polygon construction, by combining a set of half-planes using intersection and complement.)

``stroke (d, p1, p2)``
  A line of thickness ``d`` drawn from ``p1`` to ``p2``,
  with semicircle end caps of radius ``d/2``.
  Exact distance field.

``half_plane_dn (d, n)``
  A half plane with normal vector ``n``,
  whose edge is distance ``d`` from the origin.
  ``n`` must be a unit vector.
  If d >= 0, the half-plane contains the origin.
  Exact distance field.

``half_plane_pn (p, n)``
  A half plane with normal vector ``n``,
  whose edge passes through point ``p``.
  ``n`` must be a unit vector.
  Exact distance field.

``half_plane_p2 (p1, p2)``
  A half-plane whose edge passes through points p1 and p2.
  Exact distance field.

``spline ???``
  TODO.
  
  * draw an open spline curve, by sweeping a circle along the curve.
  * draw a closed spline curve by filling the area it encloses.

``text font string ???``
  Draw text. TODO

3D Shapes
=========
``sphere d``
  Construct a circle of diameter ``d``, centred on the origin.

``ellipsoid (dx, dy, dz)``
  Construct an axis-aligned ellipsoid, centred on the origin,
  with width ``dx``, depth ``dy`` and height ``dz``.

``cylinder (d, h)``
  Construct a cylinder, centered on the origin, whose axis of rotation is the Z axis.
  Diameter is ``d`` and height is ``h``.

``cone (d, h)``
  Construct a cone.
  The base (of diameter ``d``) is centered on the origin.
  The apex points up, is above the origin at height ``h``.
  Axis of rotation is the Z axis.

``torus (d1, d2)``
  Construct a torus, centred on the origin, axis of rotation is Z axis.
  Major diameter is ``d1`` (center of tube to centre of tube, crossing the origin).
  Minor diameter is ``d2`` (diameter of the tube).
  Total width of shape is ``d1+d2``.

``box (dx, dy, dz)``
  Construct an axis-aligned cuboid of width ``dx``, depth ``dy`` and height ``dz``,
  centred on the origin.

``prism (n, d, h)``
  Construct a regular right prism, centred on the origin, of height ``h``.
  The base is a regular polyhedron with ``n`` sides, whose inscribed circle has diameter ``d``,
  parallel to the XY plane.

``pyramid (n, d, h)``
  Construct a regular right pyramid.
  The base is a regular polyhedron with ``n`` sides, whose inscribed circle has diameter ``d``.
  The base is embedded in the XY plane and centred on the origin.
  The apex is above the origin at height ``h``.
  TODO

``tetrahedron d``
  Construct a regular tetrahedron, centred on the origin.
  Diameter of inscribed sphere is ``d``.

``cube d``
  Construct an axis aligned cube (regular hexahedron), centred on the origin.
  Diameter of inscribed sphere (aka height of cube) is ``d``.

``octahedron d``
  Construct a regular octahedron, centred on the origin.
  Diameter of inscribed sphere is ``d``.

``dodecahedron d``
  Construct a regular dodecahedron, centred on the origin.
  Diameter of inscribed sphere is ``d``.

``icosahedron d``
  Construct a regular icosahedron, centred on the origin.
  Diameter of inscribed sphere is ``d``.

``capsule (d, p1, p2)``
  A cylinder of diameter ``d`` whose central axis extends from ``p1`` to ``p2``,
  with the addition of hemispherical end caps of radius ``d/2``.

``half_space (d, n)``
  A half-space with normal vector ``n``,
  whose face is distance ``d`` from the origin.
  
``half_space (p1, p2, p3)``
  A half-space whose face passes through points p1, p2, p3, which are not colinear.
  The normal vector is obtained from the points via the right-hand rule.
  TODO

``gyroid``

Rigid Transformations
=====================
Distance-preserving transformations of 2D and 3D shapes.

``move (dx,dy) shape``
  Translate a 2D or 3D shape across the XY plane.

``move (dx,dy,dz) shape``
  Translate a 3D shape.

``rotate angle shape``
  Rotate a 2D or 3D shape around the Z axis, counterclockwise,
  by an angle measured in radians.

``rotate (angle, axis) shape``
  Rotate a 3D shape around the specified axis, counterclockwise,
  by an angle measured in radians.

``reflect_x shape``
  Reflect a 2D/3D shape across the Y axis/YZ plane,
  mapping each point (x,y)/(x,y,z) to (-x,y)/(-x,y,z).

``at p t shape``
  Apply a transformation ``t`` to a shape,
  treating the point ``p`` as the origin point of the transformation.
  
  Example: ``square 2 >> at (1,1) (rotate(45*deg))``
  rotates the square around the point (1,1).

Non-Rigid Transformations
=========================
Non-distance-preserving transformations of 2D and 3D shapes.

``scale k shape``
  Isotropic scaling by a scale factor of ``k`` of a 2D or 3D shape.

``scale (kx, ky) shape``
  Anisotropic scaling of a 2D or 3D shape across the XY plane.

``scale (kx, ky, kz) shape``
  Anisotropic scaling of a 3D shape.

``shear ...``
  TODO

``taper ...``
  TODO

``bend ...``
  TODO

``twist d shape``
  Twist a 3D shape around the Z axis. One full revolution for each ``d`` units along the Z axis.
  Lines parallel to the Z axis will be twisted into a helix.

``shell d shape``
  Hollow out the shape, replace it by a shell of thickness ``d`` that is centred on the shape boundary.

``rect_to_polar ...``

``isosurface ...``

2D -> 3D Transformations
========================

``extrude h shape``

``pancake d shape``

``loft h shape1 shape2``
  TODO

``rotate_extrude shape``
  The half-plane defined by ``x >= 0`` is rotated 90°, mapping the +Y axis to the +Z axis.
  Then this half-plane is rotated around the Z axis, creating a solid of revolution.

``cylinder_extrude (d, d2) shape``
  An infinite strip of 2D space running along the Y axis
  and bounded by ``-d/2 <= x <= d/2``
  is wrapped into an infinite cylinder of diameter ``d2``,
  running along the Z axis and extruded towards the Z axis.
  TODO

``stereographic_extrude shape``
  The entire 2D plane is mapped onto the surface of the unit sphere
  using a stereographic projection,
  and extruded down to the origin.
  TODO

``perimeter_extrude perimeter cross_section``

3D -> 2D Transformations
========================

``slice_xy shape``

``slice_xz shape``

``slice_yz shape``

Boolean (Set Theoretic) Operations
==================================
``nothing``
  A special shape, classified as both 2D and 3D,
  that contains no geometric points.
  It's the identity element for the ``union`` operation.

``everything``
  A special infinite shape, classified as both 2D and 3D,
  that contains all geometric points.
  It's the identity element for the ``intersection`` operation.

``complement shape``
  Reverses inside and outside, so that all points inside the argument
  shape are outside the result shape, and vice versa.
  But the boundary doesn't change.
  If the input is a finite shape, the output will be infinite.

``union (shape1, shape2, ...)``
  Construct the set union of a list of zero or more shapes.
  
  The colours of shapes later in the list
  take precedence over shapes earlier in the list.
  This follows the metaphor of ``union`` as an additive operation
  where later shapes are "painted on top of" earlier shapes.

  ``union`` is an associative operation with ``nothing``
  as the identity element, meaning it is a monoid.
  The empty list is mapped to ``nothing``.
  If all of the shapes have the same colour, then
  ``union`` is commutative.

``intersection (shape1, shape2, ...)``
  Construct the set intersection of zero or more shapes.
  
  The colour of the first shape takes precedence.
  This is the opposite of the ``union`` convention.
  It follows the metaphor of ``intersection`` as a subtractive operation
  where the first shape is primary, and subsequent shapes indicate which parts of
  the primary shape not to remove.
  It is consistent with the traditional definition
  of ``difference(s1,s2)`` as ``intersection(s1,complement(s2))``.

  ``intersection`` is an associative operation.
  The empty list is mapped to ``everything``.
  If all of the shapes have the default colour,
  then ``everything`` is the identity element,
  and ``intersection`` is commutative and a monoid.
  
``difference (shape1, shape2)``
  A binary operation that subtracts shape2 from shape1,
  preserving the colour of shape1.

``symmetric_difference (shape1, shape2, ...)``
  The result contains all of the points that belong to exactly one shape in the list.
  
  This is an associative, commutative operation with ``nothing`` as its identity element.

Repetition
==========
``repeat_x d shape``

``repeat_xy d shape``

``repeat_xyz d shape``

``repeat_mirror_x shape``

``repeat_radial reps shape``

Morph and Blend
===============
Non-rigid operations for combining two shapes by "melting them together".

``morph k shape1 shape2``

``smooth_union ...``

``smooth_intersection ...``

..
  Advanced CSG Operations
  =======================
  These are expert level CSG operations that break the abstraction of a simple world of geometric shapes,
  and expose the underlying representation of shapes as Signed Distance Fields.
