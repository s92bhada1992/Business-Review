# Business-Review
Query for all Monthly Business review 
--Creating Temp Tables from executive summary 
-- Looking for weeks vs errors (Kitting and Assembly) for all dc
With Prod_Data as (Select Mapped_dc,
                          Hellofresh_week,
                          Boxes,
                          Absolute_errors,
                          Absolute_assembly_errors,
                          Absolute_kitting_errors
                   from materialized_views.prod_db_executive_summary),

    -- Extract Labour hours per dc per week
    -- Coding right names
    -- Data extracted of year 2023

     Labour_hour as (SELECT ef.hf_week         as Hellofresh_week,
                            Case ef.DC
                                when 'CAN BC' then 'Vancouver'
                                when 'AUS PER' then 'Perth'
                                when 'AUS MEL' then 'Melbourne'
                                when 'NZL' then 'Auckland'
                                when 'SW' then 'Cloudbery'
                                when 'CAN ON (T+S)' then 'Toronto'
                                when 'IT' then 'Milan'
                                when 'NO' then 'Oslo'
                                when 'NL KP' then 'Klappolder'
                                when 'NL' then 'Prismalaan'
                                when 'AUS SYD' then 'Sydney'
                                when 'CAN AB' then 'Edmonton'
                                else ef.dc end as Mapped_dc,
                            pd.Boxes,
                            pd.Absolute_errors,
                            pd.Absolute_assembly_errors,
                            pd.Absolute_kitting_errors,
                            ef.Value_add_hrs,
                            ef.none_value_add_hrs
                     FROM uploads.isa_global_efficiency ef
                             full join prod_data pd on
                                 ( Case ef.DC
                                when 'CAN BC' then 'Vancouver'
                                when 'AUS PER' then 'Perth'
                                when 'AUS MEL' then 'Melbourne'
                                when 'NZL' then 'Auckland'
                                when 'SW' then 'Cloudbery'
                                when 'CAN ON (T+S)' then 'Toronto'
                                when 'IT' then 'Milan'
                                when 'NO' then 'Oslo'
                                when 'NL KP' then 'Klappolder'
                                when 'NL' then 'Prismalaan'
                                when 'AUS SYD' then 'Sydney'
                                when 'CAN AB' then 'Edmonton'
                                else ef.dc end) = pd.Mapped_DC
                         and ef.hf_week = pd.hellofresh_week
                     where hf_week >= '2023-W01')
-- calculating Error rates,MPB , Production mPB , support dept MPB
Select  hF_month,
       Mapped_dc,
       Sum(Boxes) as Boxes_produced,
                        Sum(Absolute_errors) as errors,
                            Sum(Absolute_assembly_errors),
                            Sum(Absolute_kitting_errors),
                            Sum(Value_add_hrs),
                            Sum(none_value_add_hrs),
       Sum(Absolute_errors)*100/Sum(Boxes) as error_rate,
       Sum(Value_add_hrs)*60/Sum(boxes) as Prod_MPB,
Sum(none_value_add_hrs)*60/Sum(boxes) as suppt_MPB,
 Sum(Value_add_hrs)*60/Sum(boxes)+Sum(none_value_add_hrs)*60/Sum(boxes) as Worked_MPB
FROM (Select hellofresh_week,
concat(cast(Hellofresh_year as string),'-', cast(Hellofresh_month as string)) as hF_month
 from dimensions.date_dimension  group by 1,2 ) d
         join Labour_hour lh on d.hellofresh_week = lh.Hellofresh_week
where d.hellofresh_week >= '2023-W01'
  and lh.Mapped_dc NOT IN ('US East', 'US West', 'US Texas', 'BC')
Group by 1,2
