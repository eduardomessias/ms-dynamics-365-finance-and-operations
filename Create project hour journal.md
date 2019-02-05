ProjTable		projTable		= ProjTable::find(<ProjId>);
ProjJournalName projJournalName = ProjJournalName::find(<ProjJournalName>);
ProjCategory	projCategory	= ProjCategory::find(<ProjCategoryId>);

// Header
projJournalTable.clear();

projJournalTableData            = JournalTableData::newTable(projJournalTable);
projJournalStatic               = projJournalTableData.journalStatic();

projJournalTableData.initFromJournalName(projJournalName);

projJournalTable.JournalId      = projJournalTableData.nextJournalId();
projJournalTable.JournalType    = ProjJournalType::Hour;
projJournalTable.JournalNameId  = projJournalName.JournalNameId;

if (projJournalTable.validateWrite())
{
	projJournalTable.insert();
}
else
{
	throw Error("@ErrorLabel");
}

// Line
projJournalTrans.clear();
        
projJournalTransData.initFromJournalTable();
	
projJournalTrans.initValue();
projJournalTrans.initFromProjTable(projTable);

projJournalTrans.ProjId         = projTable.ProjId;

projJournalTrans.TransDate      = <TransDate>;
projJournalTrans.ProjTransDate  = <TransDate>;

projJournalTrans.setTransDate();

projJournalTrans.LinePropertyId = projCategory.projLinePropertyId();
projJournalTrans.CategoryId     = projCategory.CategoryId;
projJournalTrans.TaxItemGroupId = projCategory.TaxItemGroupId;

// Resource field must be in the resource view
projJournalTrans.Resource       = (select firstonly ResourceView
										where ResourceView.Worker == <Worker RecId>).RecId;

// Check if the worker is a project resource
if (projJournalTrans.Resource == 0)
{
	throw error(strFmt("@ErrorLabel", HcmWorker::find(<Worker RecId>).name())); //i.e.: Worker %1 is not a project resource
}

projJournalTrans.Worker         	= <Worker RecId>;

projJournalTrans.DefaultDimension 	= projTable.DefaultDimension;
projJournalTrans.Qty				= <Qty>;	

if (projJournalTrans.Worker)
{
	projJournalTrans.setHourPrices();
	projJournalTrans.setPeriodDate();
}

projJournalTransData.create();