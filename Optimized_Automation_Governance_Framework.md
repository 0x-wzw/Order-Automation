# OPTIMIZED AUTOMATION & GOVERNANCE FRAMEWORK

## Executive Summary

**Current State**: Manual, WhatsApp-dependent, information silos, single points of failure  
**Target State**: Automated workflows, role-based permissions, real-time visibility, audit trails  
**Investment**: RM 0 - 500/month depending on tier  
**Implementation**: 4 weeks phased rollout

---

## TIER 1: IMMEDIATE AUTOMATION (Week 1-2) - FREE

### 1.1 Google Workspace Automation Stack

#### A. Google Sheets with Apps Script (Advanced Automation)

**Enhanced Auto-Alert System**
```javascript
// Multiple trigger points for proactive management

// TRIGGER 1: Real-time stock level monitoring
function onEdit(e) {
  var sheet = e.source.getActiveSheet();
  
  // When stock movement is recorded
  if (sheet.getName() === 'Stock Movements') {
    var range = e.range;
    
    // Auto-populate timestamp
    if (range.getColumn() === 2) { // Product Code column
      var row = range.getRow();
      sheet.getRange(row, 1).setValue(new Date()); // Auto-date
      
      // Auto-fill product name
      var productCode = range.getValue();
      var productName = lookupProductName(productCode);
      sheet.getRange(row, 3).setValue(productName);
      
      // Check if this movement creates low stock situation
      checkStockThresholds(productCode);
    }
  }
}

// TRIGGER 2: Daily automated stock report
function sendDailyStockReport() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var dashboard = ss.getSheetByName('Dashboard');
  
  // Generate report
  var report = generateStockSummary();
  var lowStockItems = getLowStockItems();
  var excessStockItems = getExcessStockItems();
  
  // Build email
  var htmlBody = HtmlService.createTemplateFromFile('DailyReportTemplate');
  htmlBody.lowStock = lowStockItems;
  htmlBody.excess = excessStockItems;
  htmlBody.summary = report;
  
  // Send to stakeholders
  MailApp.sendEmail({
    to: 'manager@company.com',
    cc: 'production@company.com,logistics@company.com',
    subject: '📊 Daily Stock Report - ' + Utilities.formatDate(new Date(), 'GMT+8', 'dd/MM/yyyy'),
    htmlBody: htmlBody.evaluate().getContent()
  });
}

// TRIGGER 3: Predictive alerts (before stock runs out)
function predictStockOutage() {
  var products = getProductMaster();
  var movements = getRecentMovements(7); // Last 7 days
  
  products.forEach(function(product) {
    var avgDailyConsumption = calculateAvgConsumption(product.code, movements);
    var daysUntilStockout = product.currentStock / avgDailyConsumption;
    
    if (daysUntilStockout <= 3 && daysUntilStockout > 0) {
      sendUrgentAlert(
        'Predicted stockout for ' + product.name + ' in ' + 
        Math.round(daysUntilStockout) + ' days'
      );
    }
  });
}

// TRIGGER 4: Production planning recommendations
function suggestProductionSchedule() {
  var salesVelocity = analyzeSalesVelocity(30); // Last 30 days
  var currentStock = getCurrentStockLevels();
  var productionCapacity = getProductionCapacity();
  
  var recommendations = [];
  
  salesVelocity.forEach(function(item) {
    var optimalStock = item.avgDailySales * 14; // 2 weeks buffer
    var gap = optimalStock - currentStock[item.productCode];
    
    if (gap > 0) {
      recommendations.push({
        product: item.productName,
        recommendedQty: Math.ceil(gap),
        priority: gap > (item.avgDailySales * 7) ? 'HIGH' : 'MEDIUM',
        reason: 'Based on ' + item.avgDailySales.toFixed(1) + ' daily sales velocity'
      });
    }
  });
  
  // Send to Production Manager
  sendProductionRecommendations(recommendations);
}
```

**Governance Built-In:**
- Cell protection on formula cells (only you can edit)
- Version history (who changed what, when)
- Approval workflow for reorder point changes
- Automated backup every 24 hours

#### B. Google Forms → Automated Data Entry

**Form 1: Packing Staff Packing Confirmation**
```
https://forms.gle/[your-form]

Fields:
- Product (Dropdown - pulls from Product Master)
- Quantity Packed (Number)
- Reference PO/Invoice (Text)
- Timestamp (Auto)
- Packer Name (Auto-filled: Packing Staff)

On Submit → Apps Script:
1. Validates quantity against PO
2. Creates Stock Movement entry (OUT)
3. Notifies Logistics Manager via email/WhatsApp
4. Updates packing queue status
5. Triggers delivery notification if order complete
```

**Form 2: Production Manager Production Output**
```
Fields:
- Production Date (Date picker)
- Product (Dropdown)
- Batch Number (Auto-generated: PROD-YYYYMMDD-XX)
- Quantity Produced (Number)
- Quality Check Status (Pass/Fail)
- Notes (Optional)

On Submit → Apps Script:
1. Creates Stock Movement entry (IN)
2. Updates production log
3. Calculates production efficiency
4. Alerts if output below target
5. Updates Logistics Manager on available stock for orders
```

**Form 3: Quality Control & Wastage**
```
Fields:
- Product
- Issue Type (Damaged/Expired/Quality Fail)
- Quantity
- Root Cause (Dropdown)
- Photo Upload (Optional)

On Submit:
1. Stock adjustment (OUT - Wastage)
2. Triggers investigation workflow if wastage > 5%
3. Monthly wastage report auto-generated
```

#### C. Automated Order Acceptance Workflow

**Smart Order Processing**
```javascript
function processNewOrder(orderDetails) {
  // Step 1: Check stock availability
  var available = checkStockAvailability(orderDetails.items);
  
  if (!available.canFulfill) {
    // Option 1: Auto-decline with reason
    sendDeclineEmail(orderDetails.customer, available.shortages);
    
    // Option 2: Partial fulfillment offer
    sendPartialFulfillmentOffer(orderDetails.customer, available.canShip);
    
    // Option 3: Defer to production queue
    estimateProductionTime(available.shortages);
    sendDeferredConfirmation(orderDetails.customer, estimatedDate);
  } else {
    // Auto-reserve stock
    createStockReservation(orderDetails);
    
    // Add to packing queue
    addToPackingQueue(orderDetails);
    
    // Notify Packing Staff
    sendPackingNotification('Packing Staff', orderDetails);
    
    // Send confirmation
    sendOrderConfirmation(orderDetails.customer);
  }
}
```

**Governance:**
- Stock reservations expire after 24 hours if not packed
- Approval required for orders > RM 5,000
- Auto-escalation if packing not done within SLA

---

### 1.2 WhatsApp Business API Integration

**Setup: Use Free Alternative - Evolution API (Open Source)**

**Installation (Self-hosted on RM20/month VPS):**
```bash
# Docker-based WhatsApp gateway
docker run -d \
  --name evolution-api \
  -p 8080:8080 \
  atendai/evolution-api

# Connect to your WhatsApp number (QR code scan)
# No Twilio fees, unlimited messages
```

**Automated WhatsApp Notifications:**

1. **Stock Alerts → WhatsApp Group**
```javascript
function sendWhatsAppStockAlert(productName, currentStock, reorderPoint) {
  var message = '🚨 *STOCK ALERT*\n\n' +
                '📦 Product: ' + productName + '\n' +
                '📊 Current: ' + currentStock + ' units\n' +
                '⚠️ Reorder at: ' + reorderPoint + '\n\n' +
                '👉 Action: Coordinate production/purchase';
  
  sendWhatsApp('60123456789-group', message); // Team group
}
```

2. **Packing Notifications → Packing Staff**
```javascript
function notifyPackingDue(orderNumber, items) {
  var message = '📦 *PACKING REQUIRED*\n\n' +
                'Order: ' + orderNumber + '\n' +
                'Items:\n' +
                items.map(i => '• ' + i.name + ' x' + i.qty).join('\n') +
                '\n\nPlease pack and confirm via form.';
  
  sendWhatsApp('60129876543', message); // Packing Staff's number
}
```

3. **Delivery Updates → Logistics Manager & Customer**
```javascript
function sendDeliveryUpdate(customerNumber, orderNumber, status) {
  var message = '🚚 *DELIVERY UPDATE*\n\n' +
                'Order: ' + orderNumber + '\n' +
                'Status: ' + status + '\n' +
                'Track: [link]\n\n' +
                'Company';
  
  sendWhatsApp(customerNumber, message);
}
```

**Governance:**
- Message templates approved by you
- Rate limiting (max 100/hour to prevent spam)
- Opt-out mechanism for customers
- Audit log of all messages sent

---

### 1.3 Role-Based Access Control (RBAC)

**Google Sheets Permissions Matrix:**

| Role | Dashboard | Product Master | Stock Movements | Settings |
|------|-----------|----------------|-----------------|----------|
| **Operations Manager (Owner)** | Edit | Edit | Edit | Edit |
| **Production Manager (Production)** | View | View | Edit (IN only) | No Access |
| **Logistics Manager (Logistics)** | View | View | Edit (OUT only) | No Access |
| **Packing Staff (Packer)** | No Access | No Access | View Only | No Access |
| **Marketing Manager (Marketing)** | View | View | View | No Access |

**Implementation via Google Apps Script:**
```javascript
function onEdit(e) {
  var sheet = e.source.getActiveSheet();
  var user = Session.getActiveUser().getEmail();
  var editedColumn = e.range.getColumn();
  
  // Production Manager can only add IN movements
  if (user === 'production@company.com' && sheet.getName() === 'Stock Movements') {
    var movementType = sheet.getRange(e.range.getRow(), 4).getValue();
    if (movementType.indexOf('OUT') !== -1) {
      e.range.setValue(''); // Undo
      SpreadsheetApp.getUi().alert('You can only record IN movements (production)');
    }
  }
  
  // Logistics Manager can only add OUT movements
  if (user === 'logistics@company.com' && sheet.getName() === 'Stock Movements') {
    var movementType = sheet.getRange(e.range.getRow(), 4).getValue();
    if (movementType.indexOf('IN') !== -1 && movementType.indexOf('Purchase') === -1) {
      e.range.setValue('');
      SpreadsheetApp.getUi().alert('You can only record OUT movements (deliveries) or IN purchases');
    }
  }
  
  // Protected cells - only Operations Manager can edit reorder points
  if (sheet.getName() === 'Product Master' && editedColumn === 6) {
    if (user !== 'manager@company.com') {
      e.range.setValue(e.oldValue);
      SpreadsheetApp.getUi().alert('Only Operations Manager can change reorder points');
    }
  }
}
```

**Approval Workflows:**
```javascript
function requestApproval(action, details, requester) {
  // Create approval request
  var approvalId = createApprovalTicket(action, details, requester);
  
  // Send to approver (you)
  MailApp.sendEmail({
    to: 'manager@company.com',
    subject: '⚠️ Approval Required: ' + action,
    body: details + '\n\nApprove: [link]\nReject: [link]'
  });
  
  // Log in audit trail
  logApprovalRequest(approvalId, action, requester);
}

// Example: Large adjustment requires approval
function onStockAdjustment(productCode, quantity, reason) {
  if (Math.abs(quantity) > 50) { // Adjustment > 50 units
    requestApproval(
      'Stock Adjustment',
      'Product: ' + productCode + '\nQty: ' + quantity + '\nReason: ' + reason,
      Session.getActiveUser().getEmail()
    );
  }
}
```

---

## TIER 2: INTERMEDIATE AUTOMATION (Week 3-4) - RM 200-400/month

### 2.1 Upgrade to Zoho Inventory + Zoho CRM Integration

**Why Zoho Stack:**
- Malaysian company (good support, RM pricing)
- Affordable (RM 200/month for 3 users)
- Integrated: Orders → Inventory → Accounting → Delivery
- Mobile apps for Production Manager, Logistics Manager, Packing Staff

**Automated Workflows in Zoho:**

#### A. Sales Order → Auto-Stock Check → Packing Slip
```
New Order (Zoho CRM/Forms)
   ↓
[Auto-check stock availability]
   ↓
If Available:
   → Create Sales Order
   → Reserve stock
   → Generate packing slip
   → Email Packing Staff + add to her mobile app queue
   → Notify Logistics Manager for delivery scheduling
   
If Insufficient:
   → Create Backorder
   → Notify Production Manager (production request)
   → Auto-estimate delivery date
   → Email customer with revised timeline
```

#### B. Production Order → Auto-Stock Update
```
Production Manager scans barcode on production batch
   ↓
Mobile app records:
   - Product
   - Quantity
   - Batch number
   - Quality check
   ↓
Auto-updates:
   → Finished goods inventory
   → Raw material consumption
   → Production efficiency dashboard
   → Available to promise (ATP) for Logistics Manager
   ↓
If production completes backorder:
   → Auto-notify customer "Ready to ship"
   → Add to packing queue
```

#### C. Packing → Delivery → Invoice Automation
```
Packing Staff scans items at packing station
   ↓
System validates against packing slip
   ↓
When complete:
   → Mark order "Ready for delivery"
   → Generate shipping label
   → Notify driver via mobile app
   ↓
Driver updates status:
   → Out for delivery
   → Delivered (with photo proof)
   ↓
System auto-triggers:
   → Invoice generation
   → Email to customer
   → Stock out recorded
   → Payment follow-up workflow (if credit terms)
```

**Cost Breakdown:**
- Zoho Inventory: RM 150/month (3 users)
- Zoho CRM: RM 100/month (sales pipeline)
- Mobile scanner app: RM 50/month (or one-time RM 200 for physical scanner)

---

### 2.2 Barcode/QR System for Packing Staff Packing Station

**Setup:**
- Print QR codes for all products (free via Zoho)
- RM 150 wireless barcode scanner OR use phone camera
- Tablet at packing station (RM 400 one-time)

**Workflow:**
1. Packing Staff receives packing list on tablet
2. Scans product QR code
3. Enters quantity
4. System validates against order
5. Beeps if wrong product/quantity
6. Auto-confirms when complete
7. Prints shipping label

**Benefits:**
- Zero typing for Packing Staff
- Impossible to pack wrong item
- Real-time accuracy
- Language-independent (visual)

---

### 2.3 Automated Reordering System

**Smart Reorder Point Calculation:**
```javascript
function calculateDynamicReorderPoint(productCode) {
  var salesHistory = getSalesHistory(productCode, 90); // 90 days
  var avgDailySales = average(salesHistory);
  var stdDev = standardDeviation(salesHistory);
  var leadTime = getSupplierLeadTime(productCode); // days
  
  // Safety stock = (max daily sales - avg daily sales) × lead time
  var safetyStock = stdDev * Math.sqrt(leadTime);
  
  // Reorder point = (avg daily sales × lead time) + safety stock
  var reorderPoint = (avgDailySales * leadTime) + safetyStock;
  
  return Math.ceil(reorderPoint);
}

// Auto-adjust reorder points monthly
function updateReorderPoints() {
  var products = getAllProducts();
  
  products.forEach(function(product) {
    var newReorderPoint = calculateDynamicReorderPoint(product.code);
    var currentReorderPoint = product.reorderPoint;
    
    if (Math.abs(newReorderPoint - currentReorderPoint) > 10) {
      // Significant change detected
      logReorderPointChange(product.code, currentReorderPoint, newReorderPoint);
      
      // Update in system
      updateProductReorderPoint(product.code, newReorderPoint);
      
      // Notify you
      sendReorderPointChangeNotification(product.name, currentReorderPoint, newReorderPoint);
    }
  });
}
```

**Auto-Purchase Order Generation:**
```javascript
function autoGeneratePurchaseOrders() {
  var lowStockItems = getLowStockItems();
  var supplierOrders = {}; // Group by supplier
  
  lowStockItems.forEach(function(item) {
    var supplier = item.supplier;
    var optimalOrderQty = calculateEOQ(item); // Economic Order Quantity
    
    if (!supplierOrders[supplier]) {
      supplierOrders[supplier] = [];
    }
    
    supplierOrders[supplier].push({
      product: item.name,
      code: item.code,
      quantity: optimalOrderQty,
      unitCost: item.cost
    });
  });
  
  // Create draft POs for your review
  for (var supplier in supplierOrders) {
    var poNumber = generatePONumber();
    var totalValue = calculatePOTotal(supplierOrders[supplier]);
    
    if (totalValue < 500) {
      // Auto-send for small orders
      sendPOToSupplier(supplier, supplierOrders[supplier], poNumber);
      logAutoSentPO(poNumber, supplier, totalValue);
    } else {
      // Large orders need your approval
      createDraftPO(supplier, supplierOrders[supplier], poNumber);
      requestPOApproval(poNumber, supplier, totalValue);
    }
  }
}
```

---

## TIER 3: ADVANCED AUTOMATION (Month 2-3) - RM 400-500/month

### 3.1 Full ERP System (Odoo - Open Source)

**Why Odoo:**
- Free core (self-hosted on RM 100/month VPS)
- Or cloud hosted RM 400/month (fully managed)
- Covers: Inventory + Manufacturing + CRM + Accounting + HR
- Malaysian support available
- Scalable as you grow

**Integrated Modules:**

#### Manufacturing Module
```
Bill of Materials (BOM) Setup:
- SNK001 (Spicy Crackers) requires:
  → 0.08kg Potato (RAW001)
  → 0.02kg Seasoning (RAW002)
  → 1pc Packaging (PKG001)

When Production Manager creates Production Order for 100 SNK001:
  → System auto-reserves:
     • 8kg Potato
     • 2kg Seasoning
     • 100 Packaging bags
  → Checks raw material availability
  → If insufficient, alerts to purchase
  → Tracks work-in-progress
  → When completed, auto-consumes raw materials
  → Adds 100 SNK001 to finished goods
```

#### Quality Control Module
```
QC Checkpoints:
1. Raw material inspection (when goods received)
2. In-process inspection (during production)
3. Finished goods inspection (before packing)

Auto-workflows:
- Failed QC → Creates non-conformance report
- Auto-notifies Production Manager
- Quarantines batch
- Triggers investigation
- Statistical quality tracking
```

#### Delivery Management
```
Delivery Routes Optimization:
- Groups orders by area
- Calculates optimal route
- Assigns to driver
- Mobile app for driver:
  → Turn-by-turn navigation
  → Customer signature capture
  → Photo proof of delivery
  → Real-time ETA to customer
```

**Total Cost:**
- Odoo Cloud (3 users): RM 400/month
- OR Self-hosted: RM 100/month VPS + RM 50/month support
- Implementation: RM 3,000 one-time (optional consultant)

---

### 3.2 Business Intelligence Dashboard (Google Data Studio - FREE)

**Auto-Generated Reports:**

#### Daily Operations Dashboard
```
Metrics visible to all:
- Orders received today
- Orders packed today
- Orders delivered today
- Current stock levels (color-coded)
- Production output today
- Delivery performance (on-time %)

Updates every hour automatically.
```

#### Weekly Management Dashboard (Your view)
```
Executive KPIs:
- Revenue trend
- Inventory turnover ratio
- Stock-out incidents
- Production efficiency
- Order fulfillment rate
- Top 10 selling products
- Slow-moving inventory
- Supplier performance

Auto-emailed every Monday 9am.
```

#### Monthly Financial Dashboard
```
Accounting integration:
- Revenue by product category
- Cost of goods sold (COGS)
- Gross margin analysis
- Inventory carrying cost
- Wastage cost
- Working capital tied in inventory

Helps with financial planning & tax optimization.
```

---

### 3.3 Predictive Analytics & AI

**Demand Forecasting:**
```python
# ML model to predict next month's demand
from prophet import Prophet
import pandas as pd

def forecast_demand(product_code, periods=30):
    # Get historical sales data
    df = get_sales_history(product_code)
    
    # Prophet model (Facebook's forecasting tool)
    model = Prophet(yearly_seasonality=True,
                   weekly_seasonality=True)
    model.fit(df)
    
    # Predict next 30 days
    future = model.make_future_dataframe(periods=periods)
    forecast = model.predict(future)
    
    return forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']]

# Auto-run monthly for all products
# Feeds into production planning
# Adjusts reorder points automatically
```

**Anomaly Detection:**
```python
def detect_anomalies():
    # Unusual stock movements
    # Sudden demand spikes
    # Quality issues patterns
    # Supplier delays
    
    # Auto-alerts you to investigate
    pass
```

---

## GOVERNANCE FRAMEWORK

### 4.1 Standard Operating Procedures (SOPs)

**SOP-001: Daily Stock Reconciliation**
```
Frequency: Daily at 5pm
Owner: Production Manager
Process:
1. Physical count of high-value items
2. Compare with system stock
3. If variance > 2%, investigate
4. Record adjustment with reason
5. Operations Manager approves adjustments > 5%

Audit Trail: All reconciliations logged
```

**SOP-002: Order Acceptance Protocol**
```
Owner: Logistics Manager
Process:
1. Receive order (email/WhatsApp/call)
2. Enter in system (CRM/Zoho/Google Form)
3. System auto-checks stock
4. If available → Auto-confirm (email template sent)
5. If unavailable → Defer to Production Manager for production timeline
6. Large orders (> RM 5,000) → Escalate to Operations Manager

SLA: Response within 2 hours during business hours
```

**SOP-003: Production Planning**
```
Frequency: Weekly (every Monday)
Owner: Production Manager + Operations Manager
Process:
1. Review sales forecast (system-generated)
2. Check current stock levels
3. Review pending orders
4. Plan production for week
5. Check raw material availability
6. Create production orders in system
7. Assign to production team

Output: Weekly production schedule
```

**SOP-004: Wastage & Quality Issues**
```
Owner: Production Manager
Process:
1. Identify wastage/defect
2. Quarantine batch
3. Record in system (type, quantity, root cause)
4. If > 5% of batch → Escalate to Operations Manager
5. Take corrective action
6. Update QC checklist if process issue

Monthly Review: Wastage trends analysis
```

**SOP-005: Supplier Management**
```
Owner: Logistics Manager (with Operations Manager approval)
Process:
1. Monitor lead times in system
2. Track quality issues per supplier
3. Maintain supplier scorecard:
   - On-time delivery %
   - Quality acceptance rate
   - Price competitiveness
   - Responsiveness
4. Quarterly supplier review meeting
5. Operations Manager approves new suppliers

Target: 95% on-time delivery from suppliers
```

---

### 4.2 Access Control & Security

**Data Security Layers:**

1. **Authentication**
   - Google Workspace SSO (single sign-on)
   - 2-factor authentication mandatory
   - Password policy (12+ chars, expires every 90 days)

2. **Authorization**
   - Role-based access (as defined in RBAC matrix)
   - Principle of least privilege
   - Access review quarterly

3. **Audit**
   - All changes logged (who, what, when)
   - Critical actions require approval
   - Audit log retention: 7 years (for tax/legal)

4. **Backup**
   - Daily automated backup
   - Weekly full backup
   - Monthly off-site backup
   - Disaster recovery plan (RTO: 4 hours, RPO: 1 hour)

**Access Review Process:**
```
Quarterly (every 3 months):
1. Review all user access
2. Remove access for ex-employees
3. Adjust permissions based on role changes
4. Log review completion
5. Report to Operations Manager

Maintained by: You (or designated IT admin)
```

---

### 4.3 Performance Metrics & KPIs

**Operational KPIs (Daily Monitoring):**

| KPI | Target | Red Flag | Who Monitors |
|-----|--------|----------|--------------|
| Order fulfillment rate | >95% | <90% | Logistics Manager |
| On-time delivery | >90% | <85% | Logistics Manager |
| Stock accuracy | >98% | <95% | Production Manager |
| Production output vs. plan | >90% | <80% | Production Manager |
| Packing errors | <1% | >3% | Packing Staff/Logistics Manager |
| Stock-out incidents | <2/month | >5/month | Operations Manager |

**Financial KPIs (Weekly/Monthly):**

| KPI | Target | Formula | Review |
|-----|--------|---------|--------|
| Inventory turnover | 12x/year | COGS / Avg Inventory | Monthly |
| Days inventory outstanding | <30 days | (Inventory / COGS) × 365 | Monthly |
| Inventory carrying cost | <15% of value | (Storage + Obsolescence + Capital) / Inventory Value | Quarterly |
| Gross margin by product | >40% | (Sales - COGS) / Sales | Monthly |
| Wastage rate | <2% | Wastage Qty / Production Qty | Monthly |

**Dashboard Auto-Alerts:**
```javascript
function monitorKPIs() {
  var kpis = calculateAllKPIs();
  
  // Order fulfillment rate
  if (kpis.orderFulfillmentRate < 0.90) {
    sendAlert('CRITICAL', 'Order fulfillment dropped to ' + 
              (kpis.orderFulfillmentRate * 100).toFixed(1) + '%',
              'manager@company.com,logistics@company.com');
  }
  
  // Stock accuracy
  if (kpis.stockAccuracy < 0.95) {
    sendAlert('WARNING', 'Stock accuracy issue: ' + 
              (kpis.stockAccuracy * 100).toFixed(1) + '%',
              'production@company.com,manager@company.com');
  }
  
  // Wastage spike
  if (kpis.wastageRate > 0.05) {
    sendAlert('URGENT', 'Wastage rate elevated: ' + 
              (kpis.wastageRate * 100).toFixed(1) + '%',
              'production@company.com,manager@company.com');
  }
}

// Run every 6 hours
```

---

### 4.4 Change Management

**Process for System Changes:**

```
REQUEST → ASSESS → APPROVE → IMPLEMENT → VERIFY

1. REQUEST
   - User submits change request (form)
   - Describe: What, Why, Impact
   
2. ASSESS
   - Technical feasibility
   - Cost/benefit
   - Risk assessment
   - Timeline estimate
   
3. APPROVE
   - Operations Manager reviews
   - Approve/reject with reason
   - Prioritize if approved
   
4. IMPLEMENT
   - Develop/configure
   - Test in sandbox
   - Document changes
   - Train users
   
5. VERIFY
   - User acceptance testing
   - Monitor for issues
   - Gather feedback
   - Close or iterate
```

**Change Categories:**

| Type | Approval Level | Implementation Window |
|------|----------------|----------------------|
| Critical bug fix | Auto-approved | Immediate |
| Minor enhancement | Production Manager/Logistics Manager | Next sprint (2 weeks) |
| New feature | Operations Manager | Planned release |
| System upgrade | Operations Manager + external review | Quarterly |

---

### 4.5 Compliance & Audit Trail

**Regulatory Compliance (Malaysia):**

1. **SSM (Companies Commission)**
   - Proper accounting records (7 years retention)
   - System audit trail meets requirements

2. **LHDN (Inland Revenue Board)**
   - Inventory valuation records
   - Stock movement documentation
   - FIFO/LIFO tracking

3. **MFRS (Malaysian Financial Reporting Standards)**
   - Inventory valuation at lower of cost/NRV
   - Proper cost accounting

**Audit Trail Requirements:**

```sql
-- Every transaction must log:
CREATE TABLE audit_log (
  id PRIMARY KEY,
  timestamp DATETIME,
  user_email VARCHAR(100),
  action VARCHAR(50), -- INSERT, UPDATE, DELETE
  table_name VARCHAR(50),
  record_id VARCHAR(50),
  old_value TEXT,
  new_value TEXT,
  ip_address VARCHAR(45),
  session_id VARCHAR(100)
);

-- Cannot be deleted or modified
-- Immutable audit log
```

**Monthly Audit Checklist:**
```
□ Stock reconciliation completed (variance < 2%)
□ All stock movements have supporting docs (PO/Invoice)
□ Wastage entries have approval for >5% batches
□ Reorder point changes approved by Operations Manager
□ Supplier invoices match purchase orders
□ Production outputs match raw material consumption
□ No orphaned stock reservations (>24hrs old)
□ Backup integrity verified
□ Access log reviewed for anomalies
□ KPI dashboard reviewed with team
```

---

## IMPLEMENTATION ROADMAP

### Phase 1: Foundation (Week 1-2) - FREE
- [ ] Upload inventory tracker to Google Sheets
- [ ] Add all formulas and automation scripts
- [ ] Setup daily email alerts
- [ ] Create Google Forms for Packing Staff and Production Manager
- [ ] Implement RBAC permissions
- [ ] Train team on new system
- [ ] Run parallel with WhatsApp for 1 week
- [ ] Cut over fully to new system

**Deliverables:**
- Functioning Google Sheets system
- Team trained
- SOPs documented

### Phase 2: Automation (Week 3-4) - RM 200/month
- [ ] Setup WhatsApp Business API (Evolution API)
- [ ] Integrate WhatsApp alerts
- [ ] Implement barcode system for Packing Staff
- [ ] Setup Google Data Studio dashboards
- [ ] Add predictive stock alerts
- [ ] Implement approval workflows

**Deliverables:**
- Automated alerts via WhatsApp
- Real-time dashboards
- Packing station optimized

### Phase 3: Integration (Month 2) - RM 400/month
- [ ] Evaluate Zoho vs. Odoo
- [ ] Sign up for chosen platform
- [ ] Data migration from Google Sheets
- [ ] Configure workflows
- [ ] Setup mobile apps for team
- [ ] Integration testing
- [ ] User acceptance testing
- [ ] Go-live

**Deliverables:**
- Full ERP system operational
- End-to-end automation
- Mobile access for all

### Phase 4: Optimization (Month 3) - Ongoing
- [ ] Add demand forecasting
- [ ] Implement AI anomaly detection
- [ ] Setup advanced reporting
- [ ] Quarterly KPI review
- [ ] Continuous improvement

**Deliverables:**
- Predictive capabilities
- Data-driven decision making
- Optimized operations

---

## COST-BENEFIT ANALYSIS

### Current State Costs (Hidden/Inefficiency):

| Issue | Annual Cost (Est.) |
|-------|-------------------|
| Stock-outs (lost sales) | RM 10,000 |
| Overstock (carrying cost) | RM 5,000 |
| Wastage (poor tracking) | RM 3,000 |
| Order errors (rework) | RM 2,000 |
| Time wasted (manual work) | RM 15,000 |
| **TOTAL** | **RM 35,000/year** |

### New System Costs:

| Tier | Monthly | Annual | Savings |
|------|---------|--------|---------|
| Tier 1 (Google) | RM 0 | RM 0 | RM 15,000 |
| Tier 2 (Automation) | RM 250 | RM 3,000 | RM 25,000 |
| Tier 3 (ERP) | RM 500 | RM 6,000 | RM 35,000 |

**ROI:**
- Tier 1: Infinite (free, saves RM 15K)
- Tier 2: 733% (spend RM 3K, save RM 25K)
- Tier 3: 483% (spend RM 6K, save RM 35K)

**Payback Period:**
- Tier 1: Immediate
- Tier 2: 2 months
- Tier 3: 3 months

---

## GOVERNANCE QUICK REFERENCE

### Daily Checklist (5 minutes)
```
☑ Check Dashboard for red alerts
☑ Review pending approvals (if any)
☑ Glance at KPI dashboard
☑ Check team has logged their work
```

### Weekly Review (30 minutes)
```
☑ Review KPI trends
☑ Check stock accuracy reconciliation
☑ Review production vs. plan
☑ Check supplier performance
☑ Review wastage reports
☑ Plan next week's production with Production Manager
```

### Monthly Strategic Review (2 hours)
```
☑ Financial performance review
☑ Inventory turnover analysis
☑ Demand forecast review
☑ Supplier scorecard review
☑ System usage audit
☑ Security access review
☑ Process improvement discussions
☑ Update SOPs if needed
```

### Quarterly Business Review (Half day)
```
☑ Strategic KPI review
☑ System capability assessment
☑ Technology roadmap update
☑ Team training needs
☑ Budget vs. actual
☑ Customer feedback analysis
☑ Competitive landscape
☑ Next quarter planning
```

---

## RISK MITIGATION

### Key Risks & Countermeasures:

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| System downtime | Low | High | Daily backups, 4-hour recovery SLA |
| User resistance | Medium | Medium | Training, change management, pilot period |
| Data loss | Low | Critical | Automated backups, 7-year retention |
| Incorrect stock data | Medium | High | Daily reconciliation, audit trails |
| Over-automation (losing human oversight) | Low | Medium | Approval gates, anomaly alerts |
| Vendor lock-in | Low | Low | Choose open-source or exportable platforms |

---

## SUCCESS METRICS (90 Days Post-Implementation)

**Target Outcomes:**

✅ **Operational Excellence:**
- 95%+ order fulfillment rate
- <2 hours order response time
- <1% packing errors
- 98%+ stock accuracy

✅ **Team Efficiency:**
- 50% reduction in time spent on manual stock tracking
- Zero WhatsApp dependency for operations
- Packing Staff can work independently (no Logistics Manager bottleneck)
- Real-time visibility for all stakeholders

✅ **Financial Performance:**
- 20% reduction in stock-outs
- 30% reduction in excess inventory
- 50% reduction in wastage
- RM 10,000+ cost savings in first 6 months

✅ **Governance & Control:**
- 100% audit trail compliance
- Zero unauthorized stock movements
- Monthly reconciliation <2% variance
- All SOPs documented and followed

---

## NEXT STEPS

**Immediate Actions (This Week):**

1. **Approve this framework** (or provide feedback for revision)
2. **Decide on tier** (recommend starting with Tier 1, upgrade after 1 month)
3. **Schedule kickoff meeting** with Production Manager, Logistics Manager, Packing Staff (1 hour)
4. **Assign project owner** (recommend Production Manager as deputy to you)
5. **Set go-live date** (recommend 2 weeks from now)

**I can help you with:**

A. Create the Google Forms for Packing Staff and Production Manager  
B. Write all the Apps Script automation code  
C. Setup WhatsApp integration (Evolution API)  
D. Create SOPs and training materials  
E. Build the Data Studio dashboards  
F. Design the implementation project plan (Gantt chart)  
G. Setup Zoho/Odoo evaluation criteria

**Which would you like me to tackle first?**
