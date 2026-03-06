# Building Operations Agent

## 🎯 Objective
สร้าง Operations Agent สำหรับ automotive business ที่จัดการ inventory, logistics, service scheduling และ operational tasks

---

## 🏭 Operations Agent Architecture

### Core Capabilities

Operations Agent ควรมี:
- **Inventory Management**: Track vehicle stock, parts inventory
- **Logistics Coordination**: Manage delivery, transport
- **Service Scheduling**: Book service appointments, track maintenance
- **Quality Control**: Inspect vehicles, document checks
- **Supply Chain**: Manage supplier orders, stock levels
- **Performance Metrics**: Track operational efficiency

### Agent Structure

```javascript
class OperationsAgent extends BasicAgent {
  constructor() {
    super('OperationsAgent');
    this.inventoryManager = new InventoryManager();
    this.logisticsEngine = new LogisticsEngine();
    this.serviceScheduler = new ServiceScheduler();
    this.qualityControl = new QualityControlSystem();
    this.metrics = new OperationsMetrics();
  }

  async respond(message, context = {}) {
    const intent = await this.analyzeOperationalIntent(message);
    const operationalContext = await this.getOperationalContext(context);
    const response = await this.generateOperationalResponse(message, intent, operationalContext);

    await this.logOperation(message, response, intent);

    return response;
  }

  async analyzeOperationalIntent(message) {
    const prompt = `Analyze this operational inquiry:

Message: "${message}"

Possible intents:
- inventory_check: Check vehicle or parts stock
- stock_replenishment: Order more vehicles/parts
- delivery_scheduling: Schedule vehicle delivery
- service_booking: Schedule maintenance/service
- quality_inspection: Vehicle inspection request
- logistics_tracking: Track shipment status
- performance_report: Request operational metrics
- supplier_coordination: Supplier management

Return JSON: {"intent": "intent_type", "confidence": 0-1}`;

    const analysis = await this.think(prompt);
    return JSON.parse(analysis);
  }
}
```

---

## 📦 Inventory Management System

### Stock Tracking

```javascript
class InventoryManager {
  constructor() {
    this.vehicleInventory = new Map();
    this.partsInventory = new Map();
  }

  async checkVehicleInventory(vehicleModel = null) {
    if (vehicleModel) {
      return await this.getVehicleStock(vehicleModel);
    }

    const inventory = await this.getAllVehicles();
    return inventory;
  }

  async getVehicleStock(model) {
    const vehicles = await this.queryDatabase(`
      SELECT 
        model, color, status, location, created_at, price
      FROM vehicles
      WHERE model = $1 AND status IN ('available', 'ready_for_sale')
      ORDER BY created_at DESC
    `, [model]);

    return {
      model,
      totalStock: vehicles.length,
      byColor: this.groupByColor(vehicles),
      byLocation: this.groupByLocation(vehicles),
      vehicles
    };
  }

  groupByColor(vehicles) {
    const grouped = {};

    vehicles.forEach(v => {
      grouped[v.color] = (grouped[v.color] || 0) + 1;
    });

    return grouped;
  }

  groupByLocation(vehicles) {
    const grouped = {};

    vehicles.forEach(v => {
      grouped[v.location] = (grouped[v.location] || 0) + 1;
    });

    return grouped;
  }

  async getAllVehicles() {
    return await this.queryDatabase(`
      SELECT model, COUNT(*) as count, status, location
      FROM vehicles
      WHERE status IN ('available', 'ready_for_sale')
      GROUP BY model, status, location
      ORDER BY count DESC
    `);
  }

  formatInventoryReport(inventory) {
    let report = `📊 Vehicle Inventory Report\n`;
    report += `════════════════════════════\n\n`;

    if (inventory.totalStock !== undefined) {
      // Single model report
      report += `🚗 ${inventory.model}\n`;
      report += `Total Stock: ${inventory.totalStock} units\n\n`;

      report += `By Color:\n`;
      Object.entries(inventory.byColor).forEach(([color, count]) => {
        report += `  • ${color}: ${count}\n`;
      });

      report += `\nBy Location:\n`;
      Object.entries(inventory.byLocation).forEach(([location, count]) => {
        report += `  • ${location}: ${count}\n`;
      });
    } else {
      // All vehicles report
      report += `Total Models: ${inventory.length}\n\n`;

      inventory.slice(0, 10).forEach(item => {
        report += `${item.model}: ${item.count} units (${item.location})\n`;
      });
    }

    return report;
  }
}
```

### Stock Replenishment Logic

```javascript
class StockReplenishment {
  async replenishStock(model, quantity, expectedDelivery) {
    const order = {
      orderId: this.generateOrderId(),
      model,
      quantity,
      expectedDelivery,
      status: 'pending',
      createdAt: new Date(),
      createdBy: 'system'
    };

    // Save to database
    await this.saveOrder(order);

    // Notify suppliers
    const suppliers = await this.getSuppliers(model);
    await this.notifySuppliers(suppliers, order);

    return {
      orderId: order.orderId,
      model,
      quantity,
      status: 'Order placed',
      expectedDelivery,
      message: `Order ${order.orderId} placed for ${quantity} ${model} units.\nExpected delivery: ${expectedDelivery}`
    };
  }
}
```

---

## 🚚 Logistics & Delivery Management

### Delivery Scheduling

```javascript
class LogisticsEngine {
  constructor() {
    this.deliverySchedule = [];
    this.carriers = [];
  }

  async scheduleDelivery(vehicleId, destination, customerInfo) {
    // Check vehicle availability
    const vehicle = await this.getVehicle(vehicleId);

    if (!vehicle) {
      return { success: false, error: 'Vehicle not found' };
    }

    // Find optimal carrier
    const carriers = await this.findAvailableCarriers(
      vehicle.currentLocation,
      destination,
      new Date()
    );

    if (carriers.length === 0) {
      return {
        success: false,
        error: 'No carriers available for this route',
        alternativeDate: this.suggestAlternativeDate()
      };
    }

    // Select best carrier (price, speed, rating)
    const selectedCarrier = this.selectOptimalCarrier(carriers);

    // Create delivery record
    const delivery = {
      deliveryId: this.generateDeliveryId(),
      vehicleId,
      carrier: selectedCarrier,
      destination,
      customer: customerInfo,
      estimatedDelivery: selectedCarrier.estimatedDelivery,
      cost: selectedCarrier.cost,
      status: 'scheduled',
      createdAt: new Date()
    };

    await this.saveDelivery(delivery);

    return {
      success: true,
      delivery: {
        deliveryId: delivery.deliveryId,
        carrier: selectedCarrier.name,
        estimatedDelivery: delivery.estimatedDelivery,
        cost: delivery.cost,
        trackingNumber: this.generateTrackingNumber()
      }
    };
  }

  async trackDelivery(deliveryId) {
    const delivery = await this.getDelivery(deliveryId);

    if (!delivery) {
      return { error: 'Delivery not found' };
    }

    const tracking = {
      deliveryId,
      status: delivery.status,
      currentLocation: delivery.currentLocation,
      estimatedDelivery: delivery.estimatedDelivery,
      progressPercent: this.calculateProgress(delivery),
      lastUpdate: delivery.lastUpdate
    };

    return tracking;
  }

  formatDeliveryInfo(delivery) {
    let info = `📦 Delivery Information\n`;
    info += `Delivery ID: ${delivery.deliveryId}\n`;
    info += `Status: ${delivery.status}\n`;
    info += `Carrier: ${delivery.carrier}\n`;
    info += `Estimated Delivery: ${delivery.estimatedDelivery}\n`;
    info += `Current Location: ${delivery.currentLocation || 'Warehouse'}\n`;

    return info;
  }
}
```

---

## 🔧 Service Scheduling & Maintenance

### Service Appointment Booking

```javascript
class ServiceScheduler {
  constructor() {
    this.technicians = [];
    this.serviceTypes = {
      'routine_maintenance': { duration: 1, cost: 2000 },
      'oil_change': { duration: 0.5, cost: 1500 },
      'inspection': { duration: 1, cost: 1000 },
      'warranty_service': { duration: 2, cost: 0 },
      'repair': { duration: 3, cost: 'quote' }
    };
  }

  async bookService(vehicleId, serviceType, preferredDate, customerContacts) {
    // Validate service type
    if (!this.serviceTypes[serviceType]) {
      return { success: false, error: 'Invalid service type' };
    }

    // Find available slots
    const availability = await this.findAvailableSlots(
      serviceType,
      preferredDate
    );

    if (availability.length === 0) {
      const alternatives = await this.findAlternativeDates(serviceType);
      return {
        success: false,
        error: 'No availability on preferred date',
        alternatives
      };
    }

    // Book first available slot
    const slot = availability[0];

    const appointment = {
      appointmentId: this.generateAppointmentId(),
      vehicleId,
      serviceType,
      technician: slot.technician,
      date: slot.date,
      time: slot.time,
      duration: this.serviceTypes[serviceType].duration,
      estimatedCost: this.serviceTypes[serviceType].cost,
      customerContact: customerContacts,
      status: 'confirmed',
      createdAt: new Date()
    };

    await this.saveAppointment(appointment);

    return {
      success: true,
      appointment: {
        appointmentId: appointment.appointmentId,
        date: appointment.date,
        time: appointment.time,
        technician: appointment.technician,
        estimatedCost: appointment.estimatedCost,
        duration: appointment.duration
      }
    };
  }

  async findAvailableSlots(serviceType, preferredDate) {
    const duration = this.serviceTypes[serviceType].duration;

    // Get all technicians
    const technicians = await this.getTechnicians();

    // Find slots for each technician
    const slots = [];

    for (const tech of technicians) {
      const schedule = await this.getTechnicianSchedule(tech.id, preferredDate);
      const freeSlots = this.findFreeSlots(schedule, duration);

      slots.push(
        ...freeSlots.map(slot => ({
          technician: tech.name,
          date: preferredDate,
          time: slot.time,
          endTime: slot.endTime
        }))
      );
    }

    return slots.sort((a, b) => a.time.localeCompare(b.time));
  }

  formatServiceAppointment(appointment) {
    let info = `🔧 Service Appointment Confirmed\n`;
    info += `════════════════════════════════\n\n`;
    info += `📋 Appointment ID: ${appointment.appointmentId}\n`;
    info += `📅 Date: ${appointment.date}\n`;
    info += `⏰ Time: ${appointment.time}\n`;
    info += `👨‍🔧 Technician: ${appointment.technician}\n`;
    info += `🛠️ Service: ${appointment.serviceType}\n`;
    info += `⏱️ Duration: ${appointment.duration} hours\n`;
    info += `💰 Estimated Cost: ฿${appointment.estimatedCost.toLocaleString()}\n`;
    info += `\nPlease arrive 10 minutes early`;

    return info;
  }
}
```

---

## ✅ Quality Control System

### Vehicle Inspection

```javascript
class QualityControlSystem {
  async inspectVehicle(vehicleId) {
    const vehicle = await this.getVehicle(vehicleId);

    if (!vehicle) {
      return { success: false, error: 'Vehicle not found' };
    }

    const inspection = {
      checkpoints: this.runInspection(vehicle),
      photos: [],
      notes: '',
      inspector: 'system',
      timestamp: new Date()
    };

    // Run automated checks
    inspection.checkpoints = await Promise.all(
      inspection.checkpoints.map(async checkpoint => {
        const result = await this.performCheck(checkpoint, vehicle);
        return { ...checkpoint, result };
      })
    );

    // Determine pass/fail
    const passed = inspection.checkpoints.every(cp => cp.result.passed);

    return {
      success: true,
      inspectionId: this.generateInspectionId(),
      vehicleId,
      passed,
      issues: inspection.checkpoints.filter(cp => !cp.result.passed).map(cp => cp.result.issue),
      timestamp: inspection.timestamp
    };
  }

  runInspection(vehicle) {
    return [
      { checkpoint: 'Exterior Condition', category: 'visual' },
      { checkpoint: 'Engine Performance', category: 'mechanical' },
      { checkpoint: 'Tire Condition', category: 'safety' },
      { checkpoint: 'Interior Cleanliness', category: 'visual' },
      { checkpoint: 'Electronics', category: 'functional' },
      { checkpoint: 'Safety Features', category: 'safety' },
      { checkpoint: 'Documentation', category: 'admin' }
    ];
  }

  async performCheck(checkpoint, vehicle) {
    // Simulate various checks
    const checks = {
      'Exterior Condition': async () => ({
        passed: vehicle.mileage < 100000 && vehicle.condition !== 'poor',
        issue: vehicle.condition === 'poor' ? 'Poor exterior condition' : null
      }),
      'Safety Features': async () => ({
        passed: vehicle.airbags && vehicle.brakes && vehicle.lights,
        issue: !vehicle.brakes ? 'Brake system needs checking' : null
      })
    };

    const checkFn = checks[checkpoint.checkpoint];
    return checkFn ? await checkFn() : { passed: true, issue: null };
  }

  formatInspectionReport(inspection) {
    let report = `✅ Vehicle Inspection Report\n`;
    report += `════════════════════════════\n\n`;
    report += `Status: ${inspection.passed ? '✅ PASSED' : '❌ FAILED'}\n`;

    if (inspection.issues.length > 0) {
      report += `\nIssues Found:\n`;
      inspection.issues.forEach((issue, index) => {
        report += `${index + 1}. ${issue}\n`;
      });
    }

    report += `\nInspection Date: ${inspection.timestamp}\n`;

    return report;
  }
}
```

---

## 📊 Operations Metrics & Reporting

### Performance KPIs

```javascript
class OperationsMetrics {
  constructor() {
    this.metrics = {
      inventory_turnover: 0,
      delivery_on_time_rate: 0,
      service_completion_rate: 0,
      quality_pass_rate: 0,
      avg_delivery_time: 0
    };
  }

  async generateReport(period = '30days') {
    const report = {
      period,
      metrics: {},
      trends: {},
      recommendations: []
    };

    report.metrics = await this.calculateMetrics(period);
    report.trends = await this.analyzeTrends(period);
    report.recommendations = this.generateRecommendations(report.metrics);

    return report;
  }

  async calculateMetrics(period) {
    return {
      totalDeliveries: await this.countDeliveries(period),
      onTimeDeliveries: await this.countOnTimeDeliveries(period),
      serviceAppointments: await this.countServiceAppointments(period),
      qualityIssues: await this.countQualityIssues(period),
      avgDeliveryTime: await this.getAverageDeliveryTime(period)
    };
  }

  generateRecommendations(metrics) {
    const recommendations = [];

    if (metrics.onTimeDeliveries / metrics.totalDeliveries < 0.95) {
      recommendations.push('Improve delivery logistics - on-time rate below 95%');
    }

    if (metrics.qualityIssues > 5) {
      recommendations.push('Enhance quality control - too many failed inspections');
    }

    return recommendations;
  }

  formatReport(report) {
    let formatted = `📊 Operations Report (${report.period})\n`;
    formatted += `════════════════════════════════\n\n`;

    Object.entries(report.metrics).forEach(([key, value]) => {
      formatted += `${key}: ${value}\n`;
    });

    if (report.recommendations.length > 0) {
      formatted += `\n⚠️ Recommendations:\n`;
      report.recommendations.forEach((rec, i) => {
        formatted += `${i + 1}. ${rec}\n`;
      });
    }

    return formatted;
  }
}
```

---

## 🎮 Hands-on Exercises

### Exercise 1: Smart Inventory Agent

**Task:** สร้าง agent ที่ track inventory และ alert on low stock

```javascript
// Check current stock levels
// Auto-trigger orders when low
// Report inventory status
```

### Exercise 2: Delivery Coordinator

**Task:** สร้าง agent ที่ manage deliveries

```javascript
// Schedule deliveries
// Track shipments
// Notify customers of ETA
```

### Exercise 3: Service Manager

**Task:** สร้าง agent ที่ manage service bookings

```javascript
// Book service appointments
// Assign technicians
// Send service reminders
```

---

## 🚀 Best Practices

### Operations Guidelines
1. **Real-time Tracking**: Monitor inventory and deliveries
2. **Proactive Alerts**: Alert on issues before they escalate
3. **Efficiency**: Optimize routes and schedules
4. **Quality Focus**: Maintain high quality standards
5. **Communication**: Keep all stakeholders informed

---

## 🎯 Key Takeaways

1. **Operations Complexity**: Multiple interconnected systems
2. **Automation Saves Resources**: Reduce manual work
3. **Real-time Visibility**: Track everything actively
4. **Predictive Maintenance**: Prevent issues proactively
5. **Integration**: Connect all operational systems

---

## 🚀 Next Steps

Day 1 เสร็จแล้ว! ตรวจสอบ wrap-up สำหรับ summary และ preview Day 2

**พร้อมเพื่อ Day 2: Memory Systems & Multi-Agent** 🚀📚

---

*Code Examples: [exercises/day1/agents/operations-agent/](exercises/day1/agents/operations-agent/)*
