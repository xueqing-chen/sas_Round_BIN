# sas_Round_BIN
sas_Round_BIN

proc format;				
value	amt_1mon_consume_w	.-.	=1	
		0-0	=2	
		0-1200	=3	
		1200-5000	=4	
		5000-high	=5	;
value	amt_1mon_online_w	.-.	=6	
		0-800	=7	
		800-5000	=8	
		5000-high	=9	;
value	atm_bqry_succ_cnt_6mon_w	.-.	=10	
		0-5	=11	
		6-81	=12	;
value	cnt_1mon_consume_w	.-.	=13	
		0-1	=14	
		2-5	=15	
		6-high	=16	;
value	cnt_1mon_online_w	.-.	=17	
		0-1	=18	
		2-5	=19	
		6-high	=20	;
value	com_purc_amt_1mon_kh_w	.-.	=21	
		0-800	=22	
		800-4000	=23	
		4000-4500	=24	
		4500-high	=25	;
value	com_purc_cnt_1mon_kh_w	.-.	=26	
		0-1	=27	
		2-5	=28	
		6-high	=29	;
value	com_purc_fail_cnt_6mon_w	0-30	=30	
		30-237	=31	;
value	fail_cnt_3mon_kh_w	-3-15	=32	
		15-25	=33	
		25-165	=34	;
value	fail_cnt_6mon_w	-3-25	=35	
		25-45	=36	
		46-237	=37	;
value	fail_cnt_6mon_kh_w	-3-30	=38	
		30-237	=39	;
value	insuft_cnt_1mon_kh_w	0-5	=40	
		6-15	=41	
		15-75	=42	;
value	insuft_cnt_6mon_w	0-20	=43	
		20-high	=44	;
value	insuft_cnt_6mon_kh_w	0-40	=45	
		40-high	=46	;
value	n_1mon_consume_trans_type_w	.-.	=47	
		0-0	=48	
		1-1	=49	
		2-2	=50	;
value	purc_amt_1mon_kh_w	.-.	=51	
		0-800	=52	
		800-5000	=53	
		5000-high	=54	;
value	purc_debit_amt_1mon_kh_w	.-.	=55	
		0-800	=56	
		800-5600	=57	
		5600-high	=58	;
value	purc_fail_nocard_cnt_3mon_kh_w	0-10	=59	
		11-20	=60	
		21-165	=61	;
value	purc_fail_nocard_cnt_6mon_w	0-30	=62	
		30-237	=63	;
value	purcnowdfailnocardamt3mon_kh_w	0-0	=64	
		2-1000	=65	
		1000-12500	=66	
		12500-high	=67	;
value	purc_online_amt_1mon_kh_w	.-.	=68	
		0-800	=69	
		800-4000	=70	
		4000-4500	=71	
		4500-high	=72	;
value	purc_succ_nocard_cnt_1mon_kh_w	.-.	=73	
		0-1	=74	
		2-5	=75	
		6-334	=76	;


data vars;
input name $32. @;
datalines ;
amt_1mon_consume
amt_1mon_online
atm_bqry_succ_cnt_6mon
cnt_1mon_consume
cnt_1mon_online
com_purc_amt_1mon_kh
com_purc_cnt_1mon_kh
com_purc_fail_cnt_6mon
fail_cnt_3mon_kh
fail_cnt_6mon
fail_cnt_6mon_kh
insuft_cnt_1mon_kh
insuft_cnt_6mon
insuft_cnt_6mon_kh
n_1mon_consume_trans_type
purc_amt_1mon_kh
purc_debit_amt_1mon_kh
purc_fail_nocard_cnt_3mon_kh
purc_fail_nocard_cnt_6mon
purc_online_amt_1mon_kh
purc_succ_nocard_cnt_1mon_kh
purcnowdfailnocardamt3mon_kh
;
run;

proc contents data=all_vars_1 
;run;


proc sql noprint;
select compress(name||'_w=put('||name||","||name||'_w.)+0;') into: renames separated by' '
from vars;

select  compress(name||'_w')  into: woe_var_ws separated by' '
from vars where name not in ( 'target','phone') ;

quit;

%put &woe_var_ws;




data 	all_vars_2;             
set 	all_vars_fq(rename =(purc_nowd_fail_nocard_amt_3mon_k=purcnowdfailnocardamt3mon_kh));
&renames;
keep target   borrow_nid &woe_var_ws;
run;

proc contents data=all_vars_2;
run;

proc sort data=all_vars_2;
by target;run;

data all_vars_3;
set all_vars_2;
length varname $40.;
array a &woe_var_ws;
do i = 1 to dim(a);
	bin_index=a[i];
	varname=compress(scan("&woe_var_ws.",i));
	output;
end;

keep borrow_nid target   varname  bin_index;
run; 

/*查format*/

proc sql;
create table format_check as
select * from all_vars_3(where =(bin_index<1));
quit;



%let total_target=116;
%let total_noevent=2901;
proc sql;
create table bin as
select bin_index, count(*) as total_cnt, sum(target) as event_cnt, 
sum(1-target) as NonEvent_Cnt,log(sum(target) /&total_target/(sum(1-target)/&total_noevent)) as WOE ,
(sum(target) /&total_target-(sum(1-target)/&total_noevent))*log(sum(target) /&total_target/(sum(1-target)/&total_noevent)) as IV
from all_vars_3 group by bin_index;
quit;
