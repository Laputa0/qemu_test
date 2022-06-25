# qemu numa test

qemu guest numa test

- kernel document
[kernel html doc](https://www.kernel.org/doc/html/latest/)

- numa doc
[numa doc](https://www.kernel.org/doc/Documentation/devicetree/bindings/numa.txt)

from doc
> ==============================================================================
3 - distance-map
==============================================================================

The optional device tree node distance-map describes the relative
distance (memory latency) between all numa nodes.

- compatible : Should at least contain "numa-distance-map-v1".

- distance-matrix
  This property defines a matrix to describe the relative distances
  between all numa nodes.
  It is represented as a list of node pairs and their relative distance.

  Note:
	1. Each entry represents distance from first node to second node.
	The distances are equal in either direction.
	2. The distance from a node to self (local distance) is represented
	with value 10 and all internode distance should be represented with
	a value greater than 10.
	3. distance-matrix should have entries in lexicographical ascending
	order of nodes.
	4. There must be only one device node distance-map which must
	reside in the root node.
	5. If the distance-map node is not present, a default
	distance-matrix is used.


- qemu source
```c
//qemu/hw/core/numa.c

void parse_numa_distance(MachineState *ms, NumaDistOptions *dist, Error **errp)
{
    uint16_t src = dist->src;
    uint16_t dst = dist->dst;
    uint8_t val = dist->val;
    NodeInfo *numa_info = ms->numa_state->nodes;

    if (src >= MAX_NODES || dst >= MAX_NODES) {
        error_setg(errp, "Parameter '%s' expects an integer between 0 and %d",
                   src >= MAX_NODES ? "src" : "dst", MAX_NODES - 1);
        return;
    }

    if (!numa_info[src].present || !numa_info[dst].present) {
        error_setg(errp, "Source/Destination NUMA node is missing. "
                   "Please use '-numa node' option to declare it first.");
        return;
    }

    if (val < NUMA_DISTANCE_MIN) {
        error_setg(errp, "NUMA distance (%" PRIu8 ") is invalid, "
                   "it shouldn't be less than %d.",
                   val, NUMA_DISTANCE_MIN);
        return;
    }

    if (src == dst && val != NUMA_DISTANCE_MIN) {
        error_setg(errp, "Local distance of node %d should be %d.",
                   src, NUMA_DISTANCE_MIN);
        return;
    }

    numa_info[src].distance[dst] = val;
    ms->numa_state->have_numa_distance = true;
}
```
