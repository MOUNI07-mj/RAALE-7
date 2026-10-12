# AI-Based Warehouse Space and Material Optimization

## 1. Problem Understanding

Modern warehouses face a persistent challenge: as SKU counts grow and order velocities
diverge, items that are picked frequently often end up in deep, hard-to-reach storage
slots while low-demand goods occupy the prime real estate near the packing station.
This mismatch inflates picker travel time, increases labor costs, and slows order
fulfillment.

The core problem is **storage-slot assignment** — deciding which product goes into
which physical location so that the total picking travel distance is minimized, while
still respecting the volume capacity of every slot and ensuring every item has a home.

### Key Constraints
- Each storage location has a fixed volumetric capacity (L x W x H).
- Each product occupies volume based on its physical dimensions.
- A product can only be assigned to a location whose remaining capacity is sufficient.
- One product per slot allocation step (greedy first-fit).
- Picking origin is a single centralized packing station.

---

## 2. Methodology

The optimization engine uses a **greedy priority-based assignment** algorithm:

### Step 1 — Priority Scoring
Every product receives a composite priority score:

```
Score = 0.35 × DailyDemand + 0.40 × PickingFrequency + 0.25 × (MovementCount / 10)
```

- **Daily Demand** (35%): how many units are ordered per day.
- **Picking Frequency** (40%): how many times a picker visits this SKU per day
  (weighted highest — it directly drives travel distance).
- **Movement Count** (25%): historical stock movement volume, normalized.

### Step 2 — Sorting
Products are sorted descending by priority score so the most critical items are
placed first, giving them the nearest available slots.

### Step 3 — Greedy Assignment
For each product in priority order:
1. Filter storage locations to those with **sufficient remaining capacity**.
2. Sort candidates by **distance from the picking station** (ascending).
3. Assign to the nearest eligible slot and decrement its remaining capacity.

This greedy first-fit approach runs in **O(n × m)** time for *n* products and *m*
locations, making it fast enough for real-time interactive use.

### Step 4 — Distance Calculation
- **Baseline**: every product is assumed to sit at an average distance of 35 m
  from the picking station (representing a random/unoptimized layout).
- **Optimized**: each product's actual assigned distance is multiplied by its
  picking frequency to compute the optimized total travel distance.

```
TotalDistance = Σ (PickingFrequency_i × Distance_i)
```

---

## 3. Experiments

### Dataset
The prototype ships with **25 sample products** across 5 categories
(Electronics, Accessories, Home Goods, Stationery, Furniture) and **20 storage
locations** across 4 zones (A–D) with varying distances (5 m – 65 m) and capacities.

### Results

| Metric                  | Baseline       | AI Optimized   |
|-------------------------|----------------|----------------|
| Total Travel Distance   | ~35 × Σ freq   | Σ(freq × dist) |
| Avg. Distance per Pick  | 35 m           | Reduced        |
| Space Utilization       | —              | Calculated     |
| Items Successfully Placed | —            | All assignable |

### Observations
- High-frequency items (USB-C cables, screen protectors, sticky notes) are
  consistently placed in Zone A at 5–12 m from the picking station.
- Low-demand furniture items (office chairs, filing cabinets) are pushed to
  Zone D at 55–65 m, which is acceptable given their rare picking.
- The greedy approach achieves a **significant reduction** in total picking
  travel distance compared to a random baseline layout.

---

## 4. Limitations

1. **Single-objective**: the algorithm only minimizes travel distance. It does
   not consider slot height ergonomics, product weight, or hazard compatibility.
2. **Greedy, not optimal**: greedy first-fit is fast but not globally optimal.
   Metaheuristics (simulated annealing, genetic algorithms) could improve results
   by 5–15% at the cost of longer compute time.
3. **Single picker origin**: real warehouses have multiple packing stations;
   a multi-origin model would better reflect actual operations.
4. **Static demand**: the sample dataset uses fixed demand figures. A production
   system would integrate live demand forecasting.
5. **No product splitting**: each product is assigned to exactly one location.
   In practice, high-demand SKUs are often split across multiple slots for
   parallel picking.
6. **2D distance**: distance is measured as a single scalar (meters from origin)
   rather than full 3D warehouse routing with aisle constraints.

---

## 5. Future Work

- Integrate reinforcement learning for dynamic re-slotting as demand shifts.
- Add multi-objective optimization (distance + ergonomic reach + weight safety).
- Support multi-zone picker routing with congestion awareness.
- Connect to a live WMS (Warehouse Management System) API for real-time data.
- Implement batch-picking optimization to group orders by zone.
