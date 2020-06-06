# FishCube
Accuracy assessment
## Detailed information

This is the repository for the article "Micro-topographic surface reconstruction of loess slope based on point cloud data" by Yangyang Zhou and Qingfeng Zhang from Northwest A&F University.In this paper accuracy assessment algorithm program was developed with the help of C# and Visual Studio 2012. The author details are as follows:

1) Yangyang Zhou: Research Institute No.290, China National Nuclear Corporation, Shaoguan 512026, China
E-mail: yangyangerzhou@gmail.com
2) Corresponding author: Qingfeng Zhang 
College of Natural Resources & Environment, Northwest A&F University, Yangling 712100, China 
E-mail: zhqf@nwsuaf.edu.cn

## Instruction guide

In this paper we explored the moving analysis window, which has remarkable advantages in error control and reflects the local characteristics of micro-topography. 3×3, 9×9 and 16×16 moving windows were adopted to traverse the DEM pixel, which includes the original point cloud data elevation and simulated elevation. Only when each pixel of the moving windows is completely filled with both elevation information (not all the pixels include original elevation due to the lack of point caused by different slopes, rainfall intensities and erosion rills), could the corresponding accuracy be evaluated. Detailed steps are as follows:

1) Load the excel file, which has been processed from a DEM by pixel and includes the elevation information;   
2) The selected point will be set as the upper left corner of the moving analysis window, the programme will automatically filter the data without elevation information according to the size of the window; analyze the data whether it supports the window type or not. 
3) Core algorithms: if it supports, it will generate algorithm index data and all algorithms will be implemented; if it does not, the calculation will be terminated and a new point needs to be selected.

To load the data and window sampling index search in C#:

       
        {
                        
            parentFrm = _parentFrm;
        }
        public Form1()
        {
            InitializeComponent();
        }
        private DataTable dt = null;
        private System.Diagnostics.Stopwatch stopwatch = new System.Diagnostics.Stopwatch();
        /// <summary>
        /// LoadExcelData
        /// </summary>
        private void button1_Click(object sender, EventArgs e)
        {
            try
            {
                this.comboBox1.SelectedIndex = 0;

                stopwatch.Start(); //  StartToMonitorTheCode

                //_________________TheAlgorithmsNeedToBeImplemented______________________begin
                dt = DataEngineCore.GetDataTableByDataType();
                this.toolTip1.SetToolTip(this.btnReset, DataEngineCore.filePath);
                this.dataGridView1.DataSource = dt;
                //_________________TheAlgorithmsNeedToBeImplemented______________________end

                stopwatch.Stop(); //  StopMonitoring
                TimeSpan timeSpan = stopwatch.Elapsed; //  AcquireTheTotalTime
                double hours = timeSpan.TotalHours; // Hours
                double minutes = timeSpan.TotalMinutes;  // Munites
                double seconds = timeSpan.TotalSeconds;  //  Seconds
                double milliseconds = timeSpan.TotalMilliseconds;  //  Milliseconds 

                this.label1.Text = string.Format("Total:{0}Data；TimeToReadData:{3}Seconds", dt == null ? "0" : dt.Rows.Count.ToString(), hours.ToString(), minutes.ToString(), seconds.ToString("f0"), milliseconds.ToString());
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
            
        }
        /// <summary>
        /// RecoverThePresentExcelData
        /// </summary>
        private void btnReset_Click(object sender, EventArgs e)
        {
            if (dt != null)
            {
                this.dataGridView1.DataSource = dt;
                newDt = null;
                return;
            }

            this.comboBox1.SelectedIndex = 0;

        }
        /// <summary>
        /// ClearProgramData
        /// </summary>
        private void btnClear_Click(object sender, EventArgs e)
        {
            dt = null;
            this.dataGridView1.DataSource = null;
            newDt = null;

            this.comboBox1.SelectedIndex = 0;
            this.textBox1.Text = "";
            this.textBox2.Text = "";
            this.button2.Enabled = false;
            this.button3.Enabled = false;
            this.button4.Enabled = false;
        }

        #region IndexFunctionArea
        private void Form1_Load(object sender, EventArgs e)
        {
            try
            {
                this.comboBox1.SelectedIndex = 0;

                bw.DoWork += bw_DoWork;
                bw.ProgressChanged += bw_ProgressChanged;
                bw.RunWorkerCompleted += bw_RunWorkerCompleted;
                if (bw.IsBusy) return;
                bw.RunWorkerAsync();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
        }
        private void comboBox1_SelectedIndexChanged(object sender, EventArgs e)
        {
            try
            {
                if (dt == null) return;
                string rowFilterStr = "";
                if (this.comboBox1.SelectedIndex == 0)
                {

                }
                else if (this.comboBox1.SelectedIndex == 1)
                {
                    rowFilterStr = GetRowFilter(1);
                }
                else if (this.comboBox1.SelectedIndex == 2)
                {
                    rowFilterStr = GetRowFilter(2);
                }
                else if (this.comboBox1.SelectedIndex == 3)
                {
                    rowFilterStr = GetRowFilter(3);
                }
                this.dt.DefaultView.RowFilter = string.Format("{0} in ({1})",ConfigCore.fieldOBJECTID, rowFilterStr);
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
        }
        /// <summary>
        /// GenerateOBJECTIDStringMeetsTheConditionsOfSpecificSamplingWindows
        /// </summary>
        /// <param name="winType"></param>
        /// <returns></returns>
        private string GetRowFilter(int winType)
        {
            if (dt == null) return "";

            string objectidStr = "";
            string str = "";
            DataRow[] selectData = null;
            foreach (DataRow dr in dt.Rows)
            {
                if (winType == 1)
                {
                    str = VerifyWindowCore.Get3X3String(double.Parse(dr[ConfigCore.fieldRow].ToString()), double.Parse(dr[ConfigCore.fieldColumn].ToString()));
                    selectData = dt.Select(str);
                    if (selectData.Length == 9)
                    {
                        objectidStr += "," + dr[ConfigCore.fieldOBJECTID].ToString();
                    }
                }
                else if (winType == 2)
                {
                    str = VerifyWindowCore.Get9X9String(double.Parse(dr[ConfigCore.fieldRow].ToString()), double.Parse(dr[ConfigCore.fieldColumn].ToString()));
                    selectData = dt.Select(str);
                    if (selectData.Length == 81)
                    {
                        objectidStr += "," + dr[ConfigCore.fieldOBJECTID].ToString();
                    }
                }
                else if (winType == 3)
                {
                    str = VerifyWindowCore.Get16X16String(double.Parse(dr[ConfigCore.fieldRow].ToString()), double.Parse(dr[ConfigCore.fieldColumn].ToString()));
                    selectData = dt.Select(str);
                    if (selectData.Length == 256)
                    {
                        objectidStr += "," + dr[ConfigCore.fieldOBJECTID].ToString();
                    }
                }
            }

            if (string.IsNullOrEmpty(objectidStr))
            {
                objectidStr = objectidStr.Substring(1);
            }
            return objectidStr;
        }

        /// <summary>
        /// PreciselyIndexButton
        /// </summary>
        private void button5_Click(object sender, EventArgs e)
        {
            try
            {
                if (dt == null) return;
                if (string.IsNullOrEmpty(this.textBox1.Text.Trim()))
                {
                    this.dt.DefaultView.RowFilter = string.Format("");
                }
                else
                {
                    this.dt.DefaultView.RowFilter = this.textBox1.Text.Trim();
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
        }
        /// <summary>
        /// VaguelyIndexButton
        /// </summary>
        private void button9_Click(object sender, EventArgs e)
        {
            try
            {
                if (dt == null) return;
                if (string.IsNullOrEmpty(this.textBox2.Text.Trim()))
                {
                    this.dt.DefaultView.RowFilter = string.Format("");
                }
                else
                {
                    this.dt.DefaultView.RowFilter = string.Format("{1} = {0} or {2} = {0}", this.textBox2.Text.Trim(), ConfigCore.fieldRow, ConfigCore.fieldColumn);
                }
            }
            catch(Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
        } 
        #endregion

        #region SelectedRowOfDataGridViewWhetherSupportsWindowSamplingAlgorithm
        private bool flg = false;
        private void dataGridView1_SelectionChanged(object sender, EventArgs e)
        {
            if (this.dataGridView1.CurrentRow == null || dt == null) return;
            flg = true;
        }

        private void bw_DoWork(object sender, DoWorkEventArgs e)
        {
            while (true)
            {
                if (flg)
                {
                    try
                    {
                        flg = false;
                        Thread.Sleep(100);

                        if (this.dataGridView1.CurrentRow == null || dt == null) return;
                        double rowValue = double.Parse(this.dataGridView1.CurrentRow.Cells[ConfigCore.fieldRow].Value.ToString());
                        double columnValue = double.Parse(this.dataGridView1.CurrentRow.Cells[ConfigCore.fieldColumn].Value.ToString());

                        string str = "";
                        DataRow[] selectData = null;
                        str = VerifyWindowCore.Get3X3String(rowValue, columnValue);
                        selectData = dt.Select(str);

                        bw.ReportProgress(1, selectData.Length == 9);

                        str = VerifyWindowCore.Get9X9String(rowValue, columnValue);
                        selectData = dt.Select(str);

                        bw.ReportProgress(2, selectData.Length == 81);

                        str = VerifyWindowCore.Get16X16String(rowValue, columnValue);
                        selectData = dt.Select(str);

                        bw.ReportProgress(3, selectData.Length == 256);
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show(ex.Message, "Tips");
                    }
                }
            }
        }

        private void bw_RunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
        {
            //if (e.Cancelled)
            //    resultTextBox.Text += "TaskCancellation。" + "\r\n";
            //else if (e.Error != null)
            //    resultTextBox.Text += "AbnornalExistence: " + e.Error + "\r\n";
            //else
            //    resultTextBox.Text += "TaskCompletion。 " + "\r\n";
        }

        private void bw_ProgressChanged(object sender, ProgressChangedEventArgs e)
        {
            if (e.ProgressPercentage == 1)
            {
                this.button2.Enabled = bool.Parse(e.UserState.ToString());
            }
            else if (e.ProgressPercentage == 2)
            {
                this.button3.Enabled = bool.Parse(e.UserState.ToString());
            }
            else if (e.ProgressPercentage == 3)
            {
                this.button4.Enabled = bool.Parse(e.UserState.ToString());
            }
            else
            {

            }
        } 
        #endregion

        private void btnAllWin_Click(object sender, EventArgs e)
        {
            try
            {
                if (this.dataGridView1.DataSource==null || dt == null) return;
                string strContent = "";
                string str = "";
                DataRow[] selectData = dt.Select();
                AlgorithmCore alg = new AlgorithmCore();

                str = alg.GetResult(selectData);
                strContent += string.Format("FullWindowSampling,DataIndicatorsAreAsFollows：{0}{1}", Environment.NewLine, str) + Environment.NewLine;
   

                frmContent frm = new frmContent("FullWindowSampling", strContent);
                frm.ShowDialog();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
        }
        private void button2_Click(object sender, EventArgs e)
        {
            try
            {
                if (this.dataGridView1.SelectedRows == null || this.dataGridView1.SelectedRows.Count == 0 || dt == null) return;
                string strContent = "";
                double rowValue = 0;
                double columnValue = 0;
                string str = "";
                DataRow[] selectData = null;
                AlgorithmCore alg = new AlgorithmCore();
                int i = 0;
                foreach (DataGridViewRow dr in this.dataGridView1.SelectedRows)
                {
                    i++;
                    rowValue = double.Parse(dr.Cells[ConfigCore.fieldRow].Value.ToString());
                    columnValue = double.Parse(dr.Cells[ConfigCore.fieldColumn].Value.ToString());

                    str = VerifyWindowCore.Get3X3String(rowValue, columnValue);
                    selectData = dt.Select(str);
                    if (selectData.Length != 9)
                    {
                        strContent += string.Format("{0}、PointData({1},{2})DoseNotSupport3X3WindowSampling.", i.ToString(), rowValue.ToString(), columnValue.ToString()) + Environment.NewLine;
                    }
                    else
                    {
                        str = alg.GetResult(selectData);
                        strContent += string.Format("{0}、PointData({1},{2})Support3X3WindowSampling,DataIndicatorsAreAsFollows：{3}{4}", i.ToString(), rowValue.ToString(), columnValue.ToString(), Environment.NewLine, str) + Environment.NewLine;
                    }
                }
                frmContent frm = new frmContent("3X3WindowSampling", strContent);
                frm.ShowDialog();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
        }

        private void button3_Click(object sender, EventArgs e)
        {
            try
            {
                if (this.dataGridView1.SelectedRows == null || this.dataGridView1.SelectedRows.Count == 0 || dt == null) return;
                string strContent = "";
                double rowValue = 0;
                double columnValue = 0;
                string str = "";
                DataRow[] selectData = null;
                AlgorithmCore alg = new AlgorithmCore();
                int i = 0;
                foreach (DataGridViewRow dr in this.dataGridView1.SelectedRows)
                {
                    i++;
                    rowValue = double.Parse(dr.Cells[ConfigCore.fieldRow].Value.ToString());
                    columnValue = double.Parse(dr.Cells[ConfigCore.fieldColumn].Value.ToString());

                    str = VerifyWindowCore.Get9X9String(rowValue, columnValue);
                    selectData = dt.Select(str);
                    if (selectData.Length != 81)
                    {
                        strContent += string.Format("{0}、PointData({1},{2})DoseNotSupport9X9WindowSampling.", i.ToString(), rowValue.ToString(), columnValue.ToString()) + Environment.NewLine;
                    }
                    else
                    {
                        str = alg.GetResult(selectData);
                        strContent += string.Format("{0}、PointData({1},{2})Support9X9WindowSampling,DataIndicatorsAreAsFollows：{3}{4}", i.ToString(), rowValue.ToString(), columnValue.ToString(), Environment.NewLine, str) + Environment.NewLine;
                    }
                }
                frmContent frm = new frmContent("9X9WindowSampling", strContent);
                frm.ShowDialog();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
        }

        private void button4_Click(object sender, EventArgs e)
        {
            try
            {
                if (this.dataGridView1.SelectedRows == null || this.dataGridView1.SelectedRows.Count == 0 || dt == null) return;
                string strContent = "";
                double rowValue = 0;
                double columnValue = 0;
                string str = "";
                DataRow[] selectData = null;
                AlgorithmCore alg = new AlgorithmCore();
                int i = 0;
                foreach (DataGridViewRow dr in this.dataGridView1.SelectedRows)
                {
                    i++;
                    rowValue = double.Parse(dr.Cells[ConfigCore.fieldRow].Value.ToString());
                    columnValue = double.Parse(dr.Cells[ConfigCore.fieldColumn].Value.ToString());

                    str = VerifyWindowCore.Get16X16String(rowValue, columnValue);
                    selectData = dt.Select(str);
                    if (selectData.Length != 256)
                    {
                        strContent += string.Format("{0}、PointData({1},{2})DoseNotSupport16X16WindowSampling.", i.ToString(), rowValue.ToString(), columnValue.ToString()) + Environment.NewLine;
                    }
                    else
                    {
                        str = alg.GetResult(selectData);
                        strContent += string.Format("{0}、PointData({1},{2})Support16X16WindowSampling,DataIndicatorsAreAsFollows：{3}{4}", i.ToString(), rowValue.ToString(), columnValue.ToString(), Environment.NewLine, str) + Environment.NewLine;
                    }
                }
                frmContent frm = new frmContent("16X16WindowSampling", strContent);
                frm.ShowDialog();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "Tips");
            }
        }



Implement the core algorithms:

    {
        private double resultSq = 0;//Sq
        /// <summary>
        /// SqEquation
        /// </summary>
        /// <param name="selectData">Point Cloud Database</param>
        /// <returns></returns>
        public double GetJunFangGen(DataRow[] selectData)
        {
            if (selectData == null) return 0;

            double sumZ = 0;
            int dataCount = selectData.Length;
            foreach (DataRow dr in selectData)
            {
                sumZ += Math.Pow((double.Parse(dr[ConfigCore.fieldZ].ToString()) - double.Parse(dr[ConfigCore.fieldMoniZ].ToString())), 2);
            }
            resultSq = Math.Sqrt(sumZ / dataCount);

            return resultSq;
        }
        /// <summary>
        /// SakEquation
        /// </summary>
        /// <param name="selectData">Point Cloud Database</param>
        /// <param name="zType">Field Name：Z or MoniZ</param>
        /// <returns></returns>
        public double GetPianXieDu(DataRow[] selectData, string zType)
        {
            if (selectData == null || string.IsNullOrWhiteSpace(zType)) return 0;
            if (resultSq == 0)
            {
                GetJunFangGen(selectData);
            }

            double resultSak = 0;
            double sumZ = 0;
            int dataCount = selectData.Length;
            foreach (DataRow dr in selectData)
            {
                sumZ += Math.Pow(double.Parse(dr[zType].ToString()), 3);
            }
            resultSak = (sumZ / dataCount) / Math.Pow(resultSq, 3);

            return resultSak;
        }
        /// <summary>
        /// SkuEquation
        /// </summary>
        /// <param name="selectData">PointCloudDatabase</param>
        /// <param name="zType">FieldName：Z or MoniZ</param>
        /// <returns></returns>
        public double GetDouDu(DataRow[] selectData, string zType)
        {
            if (selectData == null || string.IsNullOrWhiteSpace(zType)) return 0;
            if (resultSq == 0)
            {
                GetJunFangGen(selectData);
            }

            double resultSku = 0;
            double sumZ = 0;
            int dataCount = selectData.Length;
            foreach (DataRow dr in selectData)
            {
                sumZ += Math.Pow(double.Parse(dr[zType].ToString()), 4);
            }
            resultSku = (sumZ / dataCount) / Math.Pow(resultSq, 4);

            return resultSku;
        }
        /// <summary>
        /// ReturnToTheSamplingDataIndexResults
        /// </summary>
        /// <returns></returns>
        public string GetResult(DataRow[] selectData)
        {
            if(selectData==null)return "";

            string strResult = "";
            string str1 = GetJunFangGen(selectData).ToString();
            string str4 = GetPianXieDu(selectData, ConfigCore.fieldZ).ToString();
            string str5 = GetDouDu(selectData, ConfigCore.fieldZ).ToString();
            string str8 = GetPianXieDu(selectData, ConfigCore.fieldMoniZ).ToString();
            string str9 = GetDouDu(selectData, ConfigCore.fieldMoniZ).ToString();

            strResult += string.Format("Sq：{0}" + Environment.NewLine, str1);

            strResult += string.Format("ZRealElevation：" + Environment.NewLine);
            strResult += string.Format("Sak：{0}" + Environment.NewLine, str4);
            strResult += string.Format("Sku：{0}" + Environment.NewLine, str5);

            strResult += string.Format("ZmoniSimulatedElevation：" + Environment.NewLine);
            strResult += string.Format("Sak：{0}" + Environment.NewLine, str8);
            strResult += string.Format("Sku：{0}" + Environment.NewLine, str9);

            return strResult;
        }
    }
}

## Usage

In order to describe the local characteristics of the micro-topography, six points on the none-eroded ridge and near erosion ditch were selected respectively, from the upper, middle and lower slope position.
Reproducing the classic example that you can find the test data, bellow you can see three examples calculated by the algorithm. Fig.1, different moving window sizes for Lmin and Sq (slope of 5°and SpE stage). Fig.2, relationship between Lmin and Sak under 16 × 16 moving analysis window. Fig.3, relationship between Lmin and Sku under 16 × 16 moving window. 
