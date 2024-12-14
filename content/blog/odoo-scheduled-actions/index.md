+++
title = 'Odoo Scheduled Actions'
date = '2024-12-02T22:03:36+07:00'
draft = true
description = 'Tutorial Cara Membuat Scheduled Actions Pada Odoo 17'
+++


```xml
  <record id="ir_cron_populate_leave_allocation" model="ir.cron">
      <field name="name">HR Leave Allocation: Populate Leave Allocation</field>
      <field name="model_id" ref="model_sb_leave_allocation"/>
      <field name="state">code</field>
      <field name="code">model._cron_populate_leave_alloc()</field>
      <field name="user_id" ref="base.user_root"/>
      <field name="interval_number">1</field>
      <field name="interval_type">days</field>
      <field name="numbercall">-1</field>
      <field name="nextcall" eval="(DateTime.utcnow() + timedelta(days=1)).replace(hour=18, minute=0, second=0, microsecond=0).strftime('%Y-%m-%d %H:%M:%S')"/>
      <field eval="False" name="doall"/>
  </record>
```

```python
import logging

def _cron_populate_leave_alloc(self):
  self.env.cr.execute("CALL procedure_leave_alloc()")
  self.env.cr.commit()
  _logger = logging.getLogger(__name__)
  _logger.info("Cron For Populate Leave Allocation is Running Successfully")
```