# Odoo 18 to Odoo 19 Upgrade Notes for purchase_request Module

## Summary of Changes

This document outlines all changes made to upgrade the purchase_request module from Odoo 18 to Odoo 19.

## Version Change

- **Version updated**: From `18.0.2.3.3` to `19.0.1.0.0` in `__manifest__.py`

## Critical Breaking Changes in Odoo 19

### 1. Removal of `procurement.group` Model

**Impact**: The `procurement.group` model has been completely removed from Odoo 19.

**Changes Made**:
- **File**: `models/purchase_request.py`
  - Removed the `group_id` field (Many2one to `procurement.group`)
  
- **File**: `models/stock_rule.py`
  - Removed all `group_id` logic from `_prepare_purchase_request()` method
  - Removed all `group_id` logic from `_make_pr_get_domain()` method
  
- **File**: `tests/test_purchase_request.py`
  - Removed `group_id` parameter from test setup
  - Removed test case for different procurement groups
  
- **File**: `tests/test_purchase_request_procurement.py`
  - Disabled `test_procure_purchase_request_with_fixed_group` test with explanatory note

**Business Impact**: Purchase requests can no longer be grouped by procurement groups. Alternative grouping mechanisms may need to be implemented if this functionality is critical for your business processes.

### 2. UOM (Unit of Measure) Restructuring

**Impact**: The `category_id` field on `uom.uom` model has been removed. Odoo 19 uses a hierarchical structure with `relative_uom_id` instead.

**Changes Made**:
- **File**: `models/purchase_request_line.py`
  - Removed the `product_uom_category_id` field
  - Removed the domain constraint `[('category_id', '=', product_uom_category_id)]` from `product_uom_id` field
  
**Business Impact**: UOM selection is now less restrictive. Users can select any UOM without category restrictions. The system will still handle UOM conversions correctly through the hierarchical structure.

### 3. Security Group System Restructuring

**Impact**: The `category_id` field on `res.groups` model has been replaced with `privilege_id` referencing a new `res.groups.privilege` model.

**Changes Made**:
- **File**: `security/purchase_request.xml`
  - Created new `res.groups.privilege` record
  - Updated `group_purchase_request_user` to use `privilege_id` instead of `category_id`
  - Updated `group_purchase_request_manager` to use `privilege_id` instead of `category_id`
  - Added `sequence` fields to both groups
  
**Business Impact**: No functional change, but the internal security structure follows Odoo 19's new privilege-based system.

### 4. Deprecated API Changes

**Changes Made**:
- **File**: `models/purchase_request.py`
  - Changed `self.env["res.users"].browse(self.env.uid)` to `self.env.user`
  
- **File**: `models/stock_rule.py`
  - Changed `self.env.uid` to `self.env.user.id`

**Reason**: `self.env.uid` is deprecated in Odoo 19 in favor of `self.env.user.id`

## Files Modified

1. `__manifest__.py` - Version update
2. `models/purchase_request.py` - Removed group_id field, updated deprecated API calls
3. `models/purchase_request_line.py` - Removed UOM category references
4. `models/stock_rule.py` - Removed group_id logic, updated deprecated API calls
5. `tests/test_purchase_request.py` - Updated tests to remove group_id references
6. `tests/test_purchase_request_procurement.py` - Disabled procurement group test
7. `security/purchase_request.xml` - Updated security groups to use privilege_id

## Installation

The module should now install successfully on Odoo 19. Simply:

1. Update the app list in Odoo
2. Search for "Purchase Request"
3. Click "Install"

## Known Limitations

1. **Procurement Group Management**: The ability to group purchase requests by procurement groups has been removed. This was an Odoo core feature that no longer exists in version 19.

2. **UOM Domain Restrictions**: The strict UOM category domain checking has been removed due to the restructuring of the UOM system in Odoo 19.

## Testing Recommendations

After installation, please test:

1. Creating new purchase requests
2. Adding lines with different products and UOMs
3. Converting purchase request lines to purchase orders
4. The approval workflow
5. Integration with stock moves and reordering rules

## Migration Path

If you're upgrading an existing database:

1. **Backup your database** before upgrading
2. The `group_id` field data will be lost during migration
3. Any custom code that relies on `procurement.group` will need to be refactored
4. Review any custom modules that depend on this module

## Support

For issues or questions about this upgrade, please refer to the Odoo Community Association (OCA) documentation or file an issue on the GitHub repository.

---
*Upgrade completed: October 28, 2025*
