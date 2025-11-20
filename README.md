# 📦 **Odoo Custom App – Package Log Workflow (How-To Guide)**

### _Created using Odoo Studio – Version 16/18 Compatible_

## 🚀 **Purpose**

Create a lightweight internal system to log incoming packages (not tied to inventory), track their status, capture signature/photo evidence, and confirm pickup—all without affecting stock levels.

----------

# 1. **Create the Model & Menu**

1.  Go to **Inventory** app.
    
2.  Open **Studio** → **Menus** → **Add Menu**.
    
3.  Choose **New Model**.
    
4.  Name it: **DNI Package** (or “Package Log”).
    
5.  Create the model with minimal starter fields.
    

Studio auto-creates:

-   Display field: `x_name`
    
-   Basic form and tree views
    
-   Menu item under Inventory
    

----------

# 2. **Clean the Form View**

In **Studio → Form View**:

-   Rename the auto-created top field (`x_name`) to **Name**.
    
-   Make it **not required** and **readonly**.
    
-   Delete all default fields except:
    
    -   Name (top field)
        
    -   Notes (chatter)
        

You're left with a clean canvas.

----------

# 3. **Add Core Fields**

Add the following fields:

### **Recipient**

-   Type: Many2one → `res.partner`
    
-   Technical name: `x_studio_partner_id`
    
-   Required: Yes
    
-   This links packages to employees/contractors.
    

### **Tracking Number**

-   Type: Char → `x_studio_tracking_number`
    

### **Carrier**

-   Type: Selection → UPS, USPS, FedEx, Amazon, Other
    

### **Description**

-   Type: Char
    

### **Pickup Location**

-   Type: Char
    

----------

# 4. **Add Workflow Status Field**

Add a **Selection** field (Status):

-   Technical name: `x_studio_status`
    
-   Values:
    
    -   `received` → Received
        
    -   `ready` → Ready for Pickup/Dropoff
        
    -   `picked_up` → Picked up/Delivered
        
-   Widget: **Statusbar**
    
-   Default: `received`
    

----------

# 5. **Add Delivery Metadata Fields**

Add:

### **Delivered By**

-   Type: Many2one → `res.users`
    
-   Technical name: `x_studio_delivered_by`
    
-   Readonly: Yes
    

(You reuse Odoo’s existing **Created on**, **Last Updated on**, and **Created by** fields.)

----------

# 6. **Add Proof of Delivery Fields**

### **Signature**

-   Type: Signature widget → `x_studio_signature`
    

### **Notes**

-   Type: Multiline text
    

📌 _Photos are added via chatter attachments—no multi-image field needed._

----------

# 7. **Add Workflow Buttons**

## A) **Mark Ready for Pickup/Dropoff**

1.  In Studio, click **Add a button**.
    
2.  Label: **Mark Ready for Pickup/Dropoff**
    
3.  Type: Action → Create Server Action
    
4.  Code:
    

`record.write({ 'x_studio_status': 'ready',
})
record.message_post(
    body=f"Status changed to <b>Ready for Pickup/Dropoff</b> by {env.user.name}.",
    message_type="comment",
    subtype_xmlid="mail.mt_note",
)` 

5.  Button visibility (only show when status = received):
    

Domain:

`[('x_studio_status', '!=', 'received')]` 

----------

## B) **Confirm Pickup/Dropoff**

1.  Add another button: **Confirm Pickup/Dropoff**
    
2.  Attach this server action:
    

`record.write({ 'x_studio_status': 'picked_up', 'x_studio_delivered_by': env.user.id,
})
record.message_post(
    body=f"Package marked as <b>Picked up/Delivered</b> by {env.user.name}.",
    message_type="comment",
    subtype_xmlid="mail.mt_note",
)` 

3.  Button visibility (only show when status = ready):
    

Domain:

`[('x_studio_status', '!=', 'ready')]` 

----------

# 8. **Auto-Generate the Record Name**

→ Creates names like: `John Doe - 2025-11-19 - 1Z123ABCD`

### Automated Action

-   **Model:** DNI Package
    
-   **Trigger:** After Creation
    
-   **Action Type:** Execute Python Code
    

### Code:

`for rec in records:
    recipient = rec.x_studio_partner_id.name if rec.x_studio_partner_id else  "Unknown recipient"  if rec.create_date:
        date_str = rec.create_date.strftime('%Y-%m-%d') else:
        date_str = "" tracking = rec.x_studio_tracking_number or  "" parts = [recipient] if date_str:
        parts.append(date_str) if tracking:
        parts.append(tracking)

    new_name = " - ".join(parts)

    rec.write({'x_name': new_name})` 

----------

# 9. **Add Kanban View (Grouped by Status)**

## A) Enable Kanban

In Studio → Views → **Kanban**:

-   Add fields you want to show on cards:
    
    -   Name
        
    -   Recipient
        
    -   Tracking Number
        
    -   Carrier
        

## B) Add Status to Search View

Studio → Views → Search:

-   Drag **Group By** → **Status** (`x_studio_status`)
    

## C) Set Default Grouping

In Kanban View / Search View properties:

-   Default Group By = **Status**
    

This creates columns:

`Received | Ready for  Pickup/Dropoff | Picked up/Delivered` 

Records can be dragged between columns.

----------

# 10. **(Optional but recommended) Button Logic Enhancements**

### Require Signature before pickup:

Add inside Confirm Pickup action:

`if  not rec.x_studio_signature: raise UserError("Please capture a signature before completing pickup.")` 

### Auto-archive on pickup:

`rec.write({'active': False})` 

----------

# 🎉 **End Result**

You now have a fully functional Package Logging app with:

-   Clean UI
    
-   Auto-generated package names
    
-   Signature + attachments
    
-   Buttons to move between statuses
    
-   Full chatter audit trail
    
-   Drag-and-drop Kanban workflow
    
-   Ready for Production deployment
