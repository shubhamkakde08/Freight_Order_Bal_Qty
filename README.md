# Freight_Order_Bal_Qty

CREATE OR REPLACE VIEW VIEW_BAL_FREIGHT_ORDER AS
SELECT
    /* Transporter */
    wb.trpt_remark                       AS transporter_name,

    /* Freight Order */
    fo.vrno                              AS freight_vrno,
    fo.vrdate                            AS freight_vrdate,
    fo.qtylift                           AS freight_qtylift,
    fo.rate                              AS freight_rate,
    

    /* Purchase Order */
    fo.order_vrno                        AS purchase_order_vrno,
    po.qtyorder                        AS purchase_order_qty,
    po.tolerance_qty                   AS purchase_order_tolerance,
    po.qtycancelled                       AS qty_cancelled,
    po.ORDER_DETAIL                   As Order_details,

    /* Truck */
    ge.truckno                           AS truck_no,

    /* Item */
    COALESCE(wb.item_name, ge.item_name) AS item_name,

    /* Remarks */
    wb.acc_remark                        AS acc_remark,
    wb.trpt_remark                       AS transporter_remark,
    
    
    /* Weighbridge Slip Details */
    wb.wslipno                           AS wslipno,
    wb.indate                            AS indate,
    wb.outdate                           AS outdate,

    /* Weights (WB) */
    wb.firstwt                          AS gross_wt,
    wb.secondwt                           AS tare_wt,
    wb.netwt                             AS net_wt,

    wb.partygrosswt                    AS partygross_wt,
    wb.partytearwt                   AS party_tear_wt,
    wb.partynetwt                     AS party_net_wt,

    

    /* Weighbridge (KG → MT) */
    COALESCE(SUM(wb.netwt), 0)    AS net_wt_mt,

    /* Balance Freight Qty */
    ( fo.qtylift - COALESCE(SUM(wb.netwt), 0) )
                                         AS bal_freight_order_qty
FROM
    view_freight_order fo
LEFT JOIN
    view_order_engine po
        ON fo.order_vrno = po.vrno
LEFT JOIN
    view_gatetran_engine ge
        ON fo.vrno = ge.freight_order_vrno
LEFT JOIN
    view_weighbridge_engine wb
        ON ge.vrno = wb.gate_vrno
GROUP BY
    wb.trpt_remark,
    fo.vrno,
    fo.vrdate,
    fo.qtylift,
    fo.rate,
    fo.order_vrno,
    po.qtyorder,
    po.tolerance_qty,
    po.qtycancelled,
    po.ORDER_DETAIL,
    ge.truckno,
    COALESCE(wb.item_name, ge.item_name),
    wb.wslipno,
    wb.indate,
    wb.outdate,
    wb.firstwt,
    wb.secondwt,
    wb.netwt,
    wb.partygrosswt,
    wb.partytearwt,
    wb.partynetwt,
  
    wb.acc_remark;
