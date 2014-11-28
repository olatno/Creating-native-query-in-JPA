Creating-native-query-in-JPA
============================

The purpose of java code fragment is to create a sql native query UNION statement. The senerio was, I have 3 types of account expenses namely, Paid, Prepaid and Accrued. All expenses needed to view from single interface, this implies that I need to use SQL UNION statement that was not provided by JPQL. To implememnt the native query, in the one of JPA entity class create JPA Native SQL ResultSet Mapping 
        @SqlResultSetMapping(name="resolution", 
        columns={@ColumnResult(name="PDATE"), @ColumnResult(name="ID"), @ColumnResult(name="TYPES")})
        @NamedNativeQueries({@NamedNativeQuery(name="ExpensesResolution", query="SELECT c.invoiceDate AS PDATE, b.expensesInvoiceId AS ID, c.dtype AS TYPES FROM PaidExpensesInvoice b, ExpensesInvoice c " +
        "WHERE c.expensesInvoiceId = b.expensesInvoiceId AND c.invoiceDate BETWEEN ?1 AND ?2 UNION SELECT f.paymentDate AS PDATE, f.accruedPaymentId AS ID, g.dtype AS TYPES FROM AccruedExpensesPayment f, ExpensesInvoice g " + 
        "WHERE f.expensesInvoiceId = g.expensesInvoiceId AND f.paymentDate BETWEEN ?1 AND ?2 UNION SELECT h.paymentDate AS PDATE, h.expensesPaymentId AS ID, i.dtype AS TYPES FROM PayableExpensesPayment h, ExpensesInvoice i " +
        "WHERE h.expensesInvoiceId = i.expensesInvoiceId AND h.paymentDate BETWEEN ?1 AND ?2 UNION SELECT a.paymentDate AS PDATE, a.prepaidId AS ID, d.dtype AS TYPES FROM PrepaidAllotment a, ExpensesInvoice d " +
        "WHERE a.expensesInvoiceId = d.expensesInvoiceId AND a.paymentDate BETWEEN ?1 AND ?2 ORDER BY PDATE DESC", resultSetMapping="resolution"),
        @NamedNativeQuery(name="ExpensesResolutionSupplier", query="SELECT c.invoiceDate AS PDATE, b.expensesInvoiceId AS ID, c.dtype AS TYPES FROM PaidExpensesInvoice b, ExpensesInvoice c " +
        "WHERE c.expensesInvoiceId = b.expensesInvoiceId AND c.invoiceDate BETWEEN ?1 AND ?2 AND c.supplier_supplierId = ?3 UNION SELECT f.paymentDate AS PDATE, f.accruedPaymentId AS ID, g.dtype AS TYPES FROM AccruedExpensesPayment f, ExpensesInvoice g " + 
        "WHERE f.expensesInvoiceId = g.expensesInvoiceId AND f.paymentDate BETWEEN ?1 AND ?2 AND g.supplier_supplierId = ?3 UNION SELECT h.paymentDate AS PDATE, h.expensesPaymentId AS ID, i.dtype AS TYPES FROM PayableExpensesPayment h, ExpensesInvoice i " +
        "WHERE h.expensesInvoiceId = i.expensesInvoiceId AND h.paymentDate BETWEEN ?1 AND ?2 AND i.supplier_supplierId = ?3 UNION SELECT a.paymentDate AS PDATE, a.prepaidId AS ID, d.dtype AS TYPES FROM PrepaidAllotment a, ExpensesInvoice d " +
        "WHERE a.expensesInvoiceId = d.expensesInvoiceId AND a.paymentDate BETWEEN ?1 AND ?2 AND d.supplier_supplierId = ?3 ORDER BY PDATE DESC", resultSetMapping="resolution"),
        @NamedNativeQuery(name="ExpensesResolutionInvoice", query="SELECT c.invoiceDate AS PDATE, b.expensesInvoiceId AS ID, c.dtype AS TYPES FROM PaidExpensesInvoice b, ExpensesInvoice c " +
        "WHERE c.expensesInvoiceId = b.expensesInvoiceId AND c.invoiceDate BETWEEN ?1 AND ?2 AND b.invoiceNumber = ?3 UNION SELECT f.paymentDate AS PDATE, f.accruedPaymentId AS ID, g.dtype AS TYPES FROM AccruedExpensesPayment f, ExpensesInvoice g " + 
        "WHERE f.expensesInvoiceId = g.expensesInvoiceId AND f.paymentDate BETWEEN ?1 AND ?2 AND f.invoiceNumber = ?3 UNION SELECT h.paymentDate AS PDATE, h.expensesPaymentId AS ID, i.dtype AS TYPES FROM PayableExpensesPayment h, ExpensesInvoice i, PayableExpensesInvoice p " +
        "WHERE h.expensesInvoiceId = i.expensesInvoiceId AND p.expensesInvoiceId = i.expensesInvoiceId AND h.paymentDate BETWEEN ?1 AND ?2 AND p.invoiceNumber = ?3 UNION SELECT a.paymentDate AS PDATE, a.prepaidId AS ID, d.dtype AS TYPES FROM PrepaidAllotment a, ExpensesInvoice d, PrepaidExpensesInvoice p " +
        "WHERE a.expensesInvoiceId = d.expensesInvoiceId AND p.expensesInvoiceId = d.expensesInvoiceId AND a.paymentDate BETWEEN ?1 AND ?2 AND p.invoiceNumber = ?3 ORDER BY PDATE DESC", resultSetMapping="resolution")
        })

This methods inside EJB stateless bean uses the native query 
    public byte[] expensesResolutions(String datefrom, String dateto){ 
        Query query = em.createNamedQuery("ExpensesResolution");
        query.setParameter(1, java.sql.Date.valueOf(datefrom));
        query.setParameter(2, java.sql.Date.valueOf(dateto));
 
        java.util.List<Object[]> ls = (java.util.List<Object[]>)query.getResultList();
         
        return objectWriter(multipleExpensesResolutions(ls));
    }
     
    public byte[] expensesResolutions(String datefrom, String dateto, int supplier){ 
        Query query = em.createNamedQuery("ExpensesResolutionSupplier");
        query.setParameter(1, java.sql.Date.valueOf(datefrom));
        query.setParameter(2, java.sql.Date.valueOf(dateto));
        query.setParameter(3, supplier);
         
        java.util.List<Object[]> ls = (java.util.List<Object[]>)query.getResultList();
         
        return objectWriter(multipleExpensesResolutions(ls));
    }
     
    public byte[] expensesResolutions(String datefrom, String dateto, String invoice){ 
        Query query = em.createNamedQuery("ExpensesResolutionInvoice");
        query.setParameter(1, java.sql.Date.valueOf(datefrom));
        query.setParameter(2, java.sql.Date.valueOf(dateto));
        query.setParameter(3, invoice);
         
        java.util.List<Object[]> ls = (java.util.List<Object[]>)query.getResultList();
         
        return objectWriter(multipleExpensesResolutions(ls));
    }

    public ArrayList<ArrayList<Object>> multipleExpensesResolutions(java.util.List<Object[]> ls){
         
        ArrayList<ArrayList<Object>> datas = new ArrayList<ArrayList<Object>>();
         
        for(Object[] obj : ls){
         
            PrepaidAllotment pre = null;
            PayableExpensesPayment pay = null;
            AccruedExpensesPayment sp = null;
             
            if(((String)obj[2]).equals("PaidExpensesInvoice")){
                ArrayList<Object> temp1 = new ArrayList<Object>();
                PaidExpensesInvoice paid = em.find(PaidExpensesInvoice.class, ((Integer)obj[1]).intValue());
                temp1.add("Paid");
                temp1.add(paid.getSupplier().getCompanyName());
                temp1.add(paid.getInvoiceNumber());
                temp1.add(paid.getOperationExpenses().getCodeName());
                temp1.add(paid.getInvoiceDate());
                Query querypa = em.createQuery("SELECT p FROM Posting  p WHERE p.journal=:jornal AND p.codeId.codeId = ?1");
                querypa.setParameter("jornal", paid.getJournal());
                querypa.setParameter(1, paid.getPaymentMethod().getCodeId());
                Posting paidpost = null;
                try{
                    paidpost = (Posting)querypa.getSingleResult();
                }
                catch(NoResultException ex){}
                temp1.add(paidpost.getAmount().abs());
                temp1.add(paid.getExpensesInvoiceId());
                datas.add(temp1);       
                     
            }
            if(((String)obj[2]).equals("PayableExpensesInvoice")){
         
                pay = em.find(PayableExpensesPayment.class, ((Integer)obj[1]).intValue());
                if(pay != null){
                 
                    PayableExpensesInvoice payable = em.find(PayableExpensesInvoice.class, pay.getExpensesPayableInvoice().getExpensesInvoiceId());
                 
                    ArrayList<Object> temp2 = new ArrayList<Object>();
                    Query cashquery = em.createQuery("SELECT p FROM Posting p WHERE p.journal=:jornal");                
                    cashquery.setParameter("jornal", pay.getJournal());
                    Posting post = null;
                    try{    
                        post = (Posting)cashquery.getSingleResult();
                    }
                    catch(NoResultException ex){}
                     
                        temp2.add("Payable");
                        temp2.add(payable.getSupplier().getCompanyName());
                        temp2.add(payable.getInvoiceNumber());
                        temp2.add(payable.getOperationExpenses().getCodeName());
                        temp2.add(pay.getPaymentDate());
                        temp2.add(post.getAmount().abs());
                        temp2.add(pay.getExpensesPaymentId());
                        datas.add(temp2);
                     
                     
                }
            }
             
            if(((String)obj[2]).equals("AccruedExpensesInvoice")){
                sp = em.find(AccruedExpensesPayment.class, ((Integer)obj[1]).intValue());
                if(sp != null){
                    AccruedExpensesInvoice accrued = em.find(AccruedExpensesInvoice.class, sp.getExpensesAccruedInvoice().getExpensesInvoiceId());
                    ArrayList<Object> temp3 = new ArrayList<Object>();
                    Query cashquery = em.createQuery("SELECT p FROM Posting p WHERE p.journal=:jornal");
                    cashquery.setParameter("jornal", sp.getJournal());              
                    Posting postcash = null;
                    try{    
                        postcash = (Posting)cashquery.getSingleResult();
                    }
                    catch(NoResultException ex){}
                     
                    if(postcash.getCodeId().getCodeId() != 20010){
                        temp3.add("Accrued");
                        temp3.add(accrued.getSupplier().getCompanyName());
                        temp3.add(sp.getInvoiceNumber());
                        temp3.add(accrued.getOperationExpenses().getCodeName());
                        temp3.add(sp.getPaymentDate());
                        if(postcash != null){
                            temp3.add(postcash.getAmount().abs());
                        }
 
                        temp3.add(sp.getAccruedPaymentId());
                        datas.add(temp3);
                    }
                 
                }
            }   
             
            if(((String)obj[2]).equals("PrepaidExpensesInvoice")){
             
                ArrayList<Object> temp = new ArrayList<Object>();
                pre = em.find(PrepaidAllotment.class, ((Integer)obj[1]).intValue());
                 
                if(pre != null){
 
                    PrepaidExpensesInvoice prepaid = em.find(PrepaidExpensesInvoice.class, pre.getPrepaidExpenses().getExpensesInvoiceId());
     
                    BigDecimal vatparcentage = BigDecimal.ZERO;
                    vatparcentage = getVATInclusive(prepaid.getJournal(), prepaid.getPaymentMethod().getCodeId(), 70020);
             
                    temp.add("Prepaid");
                    temp.add(prepaid.getSupplier().getCompanyName());
                    temp.add(prepaid.getInvoiceNumber());
                    temp.add(prepaid.getOperationExpenses().getCodeName());
                    temp.add(pre.getPaymentDate());
                    Query querypr = em.createQuery("SELECT p FROM Posting  p WHERE p.journal=:jornal AND p.codeId.codeId = ?1");
                    querypr.setParameter("jornal", pre.getJournal());
                    querypr.setParameter(1, prepaid.getOperationExpenses().getCodeId());
                    Posting prepaidpost = null;
                    try{
                        prepaidpost = (Posting)querypr.getSingleResult();
                    }
                    catch(NoResultException ex){}
                    BigDecimal grossvat = vatparcentage.add(new BigDecimal("100"));
                    temp.add(prepaidpost.getAmount().multiply(grossvat).divide(new BigDecimal("100"), DECIMALS, ROUNDING_MODE));
                    temp.add(pre.getPrepaidId());
                    datas.add(temp);
                }
            }
        }
 
        return datas;
     
    }
