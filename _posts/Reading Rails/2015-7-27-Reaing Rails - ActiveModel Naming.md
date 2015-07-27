---
layout: post
title: Reading Rails - ActiveModel Naming
category: Reading Rails
comments: true
---

# Reaing Rails - ActiveModel Naming

我们来看个实际例子就知道Naming模块主要是干什么了。

~~~rb
[29] pry(#<LoanApplication>)> self.class.model_name
=> #<ActiveModel::Name:0x007fe512d78440
 @collection="loan_applications",
 @element="loan_application",
 @human="Loan application",
 @i18n_key=:loan_application,
 @klass=
  LoanApplication(id: integer, borrower_id: integer, tenor: string, amount: decimal, state: string, position_id: integer, created_at: datetime, updated_at: datetime, application_id: string, residence_district_id: integer, residence_street: string, geolocation_json: text, handling_fee: decimal, interest_rate: decimal, management_fee_rate: decimal, withdrawal_fee_rate: decimal, bank_id: integer, account_number: string, applied_at: datetime, approved_at: datetime, purpose: string, deposit_fee_rate: decimal, aip_by_id: integer, bank_city_id: integer, reason_code1: string, reason_code2: string, reason_code3: string, reject_code: string, picked_up_by_id: integer, fm_score: text, bank_card_id: integer, applied_amount: integer, applied_tenor: string, push_backed_at: datetime, confirmed_at: datetime, origin: string, user_evaluation_rank: string, suspended: boolean, expected_aip_finished_at: datetime, contact_time: string, welab_product_id: integer, aip_picked_up_by_id: integer, freeze_lender_status: integer, freeze_lender_at: datetime),
 @name="LoanApplication",
 @param_key="loan_application",
 @plural="loan_applications",
 @route_key="loan_applications",
 @singular="loan_application",
 @singular_route_key="loan_application">
~~~

他根据你的类定义了collection,element,human,i18n_key等成员变量。