const {
  Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
  HeadingLevel, AlignmentType, BorderStyle, WidthType, ShadingType,
  LevelFormat, PageBreak, UnderlineType
} = require('docx');
const fs = require('fs');

const headerBorder = { style: BorderStyle.SINGLE, size: 1, color: "AAAAAA" };
const hb = { top: headerBorder, bottom: headerBorder, left: headerBorder, right: headerBorder };
const noBorder = { style: BorderStyle.NONE, size: 0, color: "FFFFFF" };
const nb = { top: noBorder, bottom: noBorder, left: noBorder, right: noBorder };

const CONTENT_W = 9360;

function h1(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_1,
    spacing: { before: 320, after: 120 },
    border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "1F3864", space: 1 } },
    children: [new TextRun({ text, bold: true, size: 28, font: "Courier New", color: "1F3864" })]
  });
}

function h2(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_2,
    spacing: { before: 240, after: 80 },
    children: [new TextRun({ text: "## " + text, bold: true, size: 24, font: "Courier New", color: "2E5496" })]
  });
}

function h3(text) {
  return new Paragraph({
    spacing: { before: 160, after: 60 },
    children: [new TextRun({ text: "### " + text, bold: true, size: 22, font: "Courier New", color: "404040" })]
  });
}

function body(text) {
  return new Paragraph({
    alignment: AlignmentType.LEFT,
    spacing: { before: 60, after: 60, line: 340 },
    children: [new TextRun({ text, size: 20, font: "Courier New" })]
  });
}

function code(text) {
  return new Paragraph({
    spacing: { before: 40, after: 40 },
    shading: { fill: "F0F0F0", type: ShadingType.CLEAR },
    indent: { left: 360 },
    children: [new TextRun({ text, size: 18, font: "Courier New", color: "333333" })]
  });
}

function bullet(text) {
  return new Paragraph({
    numbering: { reference: "bullets", level: 0 },
    spacing: { before: 40, after: 40, line: 300 },
    children: [new TextRun({ text, size: 20, font: "Courier New" })]
  });
}

function blank() {
  return new Paragraph({ spacing: { before: 40, after: 40 }, children: [new TextRun("")] });
}

function divider() {
  return new Paragraph({
    spacing: { before: 80, after: 80 },
    border: { bottom: { style: BorderStyle.SINGLE, size: 2, color: "CCCCCC", space: 1 } },
    children: [new TextRun("")]
  });
}

function makeTable(headers, rows, colWidths) {
  const total = colWidths.reduce((a, b) => a + b, 0);
  return new Table({
    width: { size: total, type: WidthType.DXA },
    columnWidths: colWidths,
    rows: [
      new TableRow({
        tableHeader: true,
        children: headers.map((h, i) => new TableCell({
          borders: hb,
          width: { size: colWidths[i], type: WidthType.DXA },
          shading: { fill: "1F3864", type: ShadingType.CLEAR },
          margins: { top: 80, bottom: 80, left: 120, right: 120 },
          children: [new Paragraph({
            alignment: AlignmentType.CENTER,
            children: [new TextRun({ text: h, bold: true, size: 18, font: "Courier New", color: "FFFFFF" })]
          })]
        }))
      }),
      ...rows.map((row, ri) => new TableRow({
        children: row.map((cell, ci) => new TableCell({
          borders: hb,
          width: { size: colWidths[ci], type: WidthType.DXA },
          shading: { fill: ri % 2 === 0 ? "F5F5F5" : "FFFFFF", type: ShadingType.CLEAR },
          margins: { top: 60, bottom: 60, left: 120, right: 120 },
          children: [new Paragraph({
            children: [new TextRun({ text: cell, size: 18, font: "Courier New" })]
          })]
        }))
      }))
    ]
  });
}

const doc = new Document({
  numbering: {
    config: [{
      reference: "bullets",
      levels: [{
        level: 0, format: LevelFormat.BULLET, text: "-",
        alignment: AlignmentType.LEFT,
        style: { paragraph: { indent: { left: 720, hanging: 360 } } }
      }]
    }]
  },
  styles: {
    default: { document: { run: { font: "Courier New", size: 20 } } },
    paragraphStyles: [
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 28, bold: true, font: "Courier New" },
        paragraph: { spacing: { before: 320, after: 120 }, outlineLevel: 0 } },
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 24, bold: true, font: "Courier New" },
        paragraph: { spacing: { before: 240, after: 80 }, outlineLevel: 1 } },
    ]
  },
  sections: [{
    properties: {
      page: {
        size: { width: 12240, height: 15840 },
        margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 }
      }
    },
    children: [

      // TITLE BLOCK
      new Paragraph({
        spacing: { before: 0, after: 0 },
        shading: { fill: "1F3864", type: ShadingType.CLEAR },
        children: [new TextRun({ text: " ", size: 8 })]
      }),
      new Paragraph({
        spacing: { before: 0, after: 0 },
        shading: { fill: "1F3864", type: ShadingType.CLEAR },
        alignment: AlignmentType.LEFT,
        children: [
          new TextRun({ text: "  README", bold: true, size: 36, font: "Courier New", color: "FFFFFF" })
        ]
      }),
      new Paragraph({
        spacing: { before: 0, after: 0 },
        shading: { fill: "1F3864", type: ShadingType.CLEAR },
        alignment: AlignmentType.LEFT,
        children: [
          new TextRun({ text: "  XAI for Bearing Fault Diagnosis", size: 22, font: "Courier New", color: "AACFEE" })
        ]
      }),
      new Paragraph({
        spacing: { before: 0, after: 200 },
        shading: { fill: "1F3864", type: ShadingType.CLEAR },
        alignment: AlignmentType.LEFT,
        children: [
          new TextRun({ text: "  Divyanshi Gupta | 2022B3A41257P | BITS Pilani | LOP — Prof. Madhurjya Dev Choudhury", size: 18, font: "Courier New", color: "AACFEE" })
        ]
      }),

      blank(),

      // OVERVIEW
      h1("Overview"),
      body("This project implements an end-to-end Explainable AI (XAI) pipeline for bearing fault"),
      body("diagnosis using the CWRU benchmark dataset. The project began with a structured"),
      body("literature review of 16 papers across four XAI methodologies — SHAP, LIME, Grad-CAM,"),
      body("and comparative multi-method studies — and progressed into full implementation covering"),
      body("signal processing, deep learning classification, and interpretability analysis."),
      blank(),
      body("The pipeline covers: raw .mat file ingestion, 1D signal extraction, STFT spectrogram"),
      body("and Cyclic Spectral Coherence (CSC) generation, ResNet50 transfer learning, Grad-CAM"),
      body("heatmap visualization, SHAP feature attribution via XGBoost, and a systematic"),
      body("comparison between STFT and CSC as CNN input representations."),

      blank(),
      h1("Dataset"),
      body("Case Western Reserve University (CWRU) Bearing Dataset"),
      body("Source : https://engineering.case.edu/bearingdatacenter"),
      body("Kaggle : https://www.kaggle.com/datasets/brjapon/cwru-bearing-datasets"),
      blank(),
      body("Bearing        : SKF 6205-2RS JEM"),
      body("Sampling rate  : 12,000 Hz"),
      body("Motor speed    : 1772 RPM (Load 1)"),
      body("Signal used    : Drive End (DE) accelerometer"),
      blank(),

      makeTable(
        ["File", "Fault Type", "Defect Size"],
        [
          ["Time_Normal_1_098.mat", "Healthy", "—"],
          ["B007_1_123.mat",        "Ball",         "0.007 in"],
          ["B014_1_190.mat",        "Ball",         "0.014 in"],
          ["B021_1_227.mat",        "Ball",         "0.021 in"],
          ["IR007_1_110.mat",       "Inner Race",   "0.007 in"],
          ["IR014_1_175.mat",       "Inner Race",   "0.014 in"],
          ["IR021_1_214.mat",       "Inner Race",   "0.021 in"],
          ["OR007_6_1_136.mat",     "Outer Race",   "0.007 in"],
          ["OR014_6_1_202.mat",     "Outer Race",   "0.014 in"],
          ["OR021_6_1_239.mat",     "Outer Race",   "0.021 in"],
        ],
        [4500, 2800, 2060]
      ),

      blank(),
      h1("Project Structure"),
      code("project/"),
      code("|"),
      code("+-- Phase 1 : Signal Processing"),
      code("|   +-- Load .mat files"),
      code("|   +-- Extract DE_time signal -> flatten to 1D array"),
      code("|   +-- Segment into overlapping windows"),
      code("|   +-- STFT spectrograms  (window=2048, step=1024)"),
      code("|   +-- CSC maps           (window=12000, step=6000)"),
      code("|"),
      code("+-- Phase 2 : Classification"),
      code("|   +-- ResNet50 backbone (pretrained ImageNet)"),
      code("|   +-- Phase 1 training : freeze base, train top (lr=1e-3, 10 epochs)"),
      code("|   +-- Phase 2 training : unfreeze last 30 layers (lr=1e-5, 10 epochs)"),
      code("|"),
      code("+-- Phase 3 : XAI"),
      code("|   +-- Grad-CAM  -> conv5_block3_out layer -> heatmap overlay"),
      code("|   +-- SHAP      -> XGBoost + TreeExplainer -> feature attribution"),
      code("|   +-- CSC maps  -> fault frequency lines overlaid (BPFO/BPFI/BSF)"),
      code("|"),
      code("+-- Phase 4 : Comparison"),
      code("    +-- stft_vs_csc_comparison.py"),
      code("        +-- Accuracy, F1, training curves, interpretability score"),

      blank(),
      h1("How to Run"),
      body("All code runs on Kaggle. Add the CWRU dataset before starting."),
      body("Do not restart the kernel between phases — all models and history objects"),
      body("must stay in memory for the comparison cell to work."),
      blank(),

      h2("Step 1 — Load and segment signals"),
      body("Load each .mat file via scipy.io.loadmat(). Extract the key containing"),
      body("'DE_time', flatten to 1D. Segment using a sliding window."),
      body("Window size: 2048 samples for STFT, 12000 samples for CSC."),
      blank(),

      h2("Step 2 — Generate representations"),
      h3("STFT"),
      body("Apply scipy.signal.stft (nperseg=128, noverlap=64). Convert complex output"),
      body("to magnitude, then dB scale (20 * log10). Normalize to [0,255]. Resize to"),
      body("224x224 via cv2.resize(). Stack 3 identical channels -> shape (224,224,3)."),
      blank(),
      h3("CSC"),
      body("Apply STFT (nperseg=512, overlap=0.75). Compute instantaneous power as |Zxx|^2."),
      body("Apply rfft along time axis to detect periodic power fluctuations. Normalize"),
      body("by mean power -> coherence values in [0,1]. Crop to cyclic 0-300 Hz and"),
      body("spectral 0-3000 Hz. Resize to 224x224. Stack 3 channels."),
      blank(),

      h2("Step 3 — Train ResNet50"),
      body("Same architecture for both representations. Build custom top:"),
      body("GlobalAveragePooling2D -> Dense(256, relu) -> Dropout(0.4) -> Dense(10, softmax)."),
      body("Save model_stft, model_csc, history_stft, history_csc before closing the kernel."),
      blank(),

      h2("Step 4 — Run Grad-CAM"),
      body("Target layer: conv5_block3_out (last conv layer of ResNet50, output 7x7x2048)."),
      body("Use GradientTape to compute gradient of class score w.r.t. feature maps."),
      body("Global average pool gradients -> importance weights. Weighted sum of feature"),
      body("maps -> ReLU -> normalize -> resize 7x7 to 224x224 -> overlay on input image."),
      blank(),

      h2("Step 5 — Run SHAP"),
      body("Load feature_time_48k_2048_load_1.csv. Train XGBoost on 9 statistical features:"),
      body("max, min, mean, sd, rms, skewness, kurtosis, crest, form."),
      body("Run shap.TreeExplainer(clf). SHAP values shape: (n_samples, 9, 10)."),
      body("Plot beeswarm (per class), bar chart (global), force plot (single sample)."),
      blank(),

      h2("Step 6 — Run comparison"),
      body("Run stft_vs_csc_comparison.py cells in order. Requires in memory:"),
      body("model_stft, model_csc, history_stft, history_csc, X_stft_test, y_stft_test,"),
      body("X_csc_test, y_csc_test. All 7 cells must run in the same session."),

      blank(),
      h1("Bearing Fault Frequencies at 1772 RPM"),
      body("Bearing geometry: N=9 balls, d=0.3126 in, D=1.748 in, contact angle=0 deg"),
      blank(),

      makeTable(
        ["Frequency", "Formula", "Value"],
        [
          ["Shaft", "RPM / 60", "29.53 Hz"],
          ["BPFO", "(N/2) x s x (1 - d/D)", "107.36 Hz"],
          ["BPFI", "(N/2) x s x (1 + d/D)", "162.18 Hz"],
          ["BSF",  "(D/2d) x s x (1 - (d/D)^2)", "141.17 Hz"],
          ["FTF",  "(s/2) x (1 - d/D)", "11.93 Hz"],
        ],
        [2200, 4000, 3160]
      ),
      blank(),
      body("These values and their harmonics (2x, 3x, 4x) are overlaid as cyan dashed lines"),
      body("on all CSC and Grad-CAM plots. Alignment of CNN activation peaks with these lines"),
      body("is the physical validation criterion used in this project."),

      blank(),
      h1("XAI Methods"),

      h2("Grad-CAM"),
      body("Model-dependent. Applied to conv5_block3_out, the last convolutional layer of"),
      body("ResNet50. Produces a spatial heatmap over the spectrogram or CSC map."),
      body("Red = high CNN attention. Validated by checking alignment with BPFO/BPFI/BSF."),
      body("Class-discriminative: different fault types activate different spatial regions."),
      blank(),

      h2("SHAP via TreeExplainer"),
      body("Model-dependent. Applied to XGBoost trained on 9 statistical features."),
      body("Produces attributions of shape (n_samples, n_features, n_classes)."),
      body("Satisfies efficiency, symmetry, and null-player axioms from game theory."),
      body("Global beeswarm shows feature importance across all samples."),
      body("Per-class plots show which features drive each specific fault type."),
      blank(),

      h2("CSC vs STFT (Representation Comparison)"),
      body("Not a standalone XAI method but an interpretability-aware design choice."),
      body("CSC encodes the fault modulation frequency directly on the horizontal axis,"),
      body("making Grad-CAM activations sharper and more physically aligned than STFT."),
      body("Evaluated using a custom Grad-CAM interpretability score:"),
      body("  score = mean activation at fault freq pixel / mean background activation"),
      body("Score > 1.0 means the CNN is attending to the physically correct region."),

      blank(),
      h1("Output Files"),

      makeTable(
        ["File", "Generated by", "Purpose"],
        [
          ["confusion_matrix_comparison.png", "Cell 2", "STFT vs CSC classification performance"],
          ["f1_comparison.png",               "Cell 3", "Per-class F1 scores side by side"],
          ["training_curves.png",             "Cell 4", "Loss and accuracy convergence curves"],
          ["gradcam_stft_vs_csc.png",         "Cell 6", "Grad-CAM with fault freq lines overlaid"],
          ["csc_maps_all_classes.png",        "CSC cell", "CSC maps for all 10 fault classes"],
          ["signals_and_spectrograms.png",    "Viz cell", "1D signal + CNN input + labeled spec"],
          ["shap_results.png",               "SHAP cell", "Feature importance, all classes"],
        ],
        [3600, 2200, 3560]
      ),

      blank(),
      h1("Key Results"),
      body("Fill these in after running Cell 7 of stft_vs_csc_comparison.py."),
      blank(),

      makeTable(
        ["Metric", "STFT", "CSC"],
        [
          ["Overall Accuracy",               "[FILL]",  "[FILL]"],
          ["Macro F1-Score",                 "[FILL]",  "[FILL]"],
          ["Grad-CAM Interpretability Score","[FILL]",  "[FILL]"],
          ["XGBoost Accuracy (SHAP branch)", "95.87%",  "—"],
          ["Top SHAP feature (global)",      "kurtosis","—"],
        ],
        [4000, 2300, 2300]
      ),

      blank(),
      h1("Dependencies"),
      body("All available by default on Kaggle. No additional installs required."),
      blank(),

      makeTable(
        ["Package", "Used for"],
        [
          ["tensorflow >= 2.x", "ResNet50 training, Grad-CAM via GradientTape"],
          ["scipy",             "STFT (scipy.signal.stft), .mat file loading"],
          ["numpy",             "All array operations"],
          ["scikit-learn",      "Train/test split, metrics, LabelEncoder"],
          ["xgboost",           "Classifier for SHAP branch"],
          ["shap",              "TreeExplainer, beeswarm and bar plots"],
          ["matplotlib",        "All visualizations"],
          ["seaborn",           "Confusion matrix heatmaps"],
          ["opencv-python",     "Image resizing (cv2.resize)"],
          ["pandas",            "CSV loading for statistical features"],
        ],
        [3800, 5560]
      ),

      blank(),
      h1("Notes"),
      bullet("Always save model_stft, model_csc, history_stft, history_csc before closing the notebook. Restarting the kernel requires full retraining."),
      bullet("CSC generates significantly fewer training samples than STFT because it requires a longer window (12000 vs 2048 samples). This is expected and should be reported as a tradeoff in the paper, not a bug."),
      bullet("The Grad-CAM interpretability score is a custom metric written for this project. It is not from any external library. The formula is documented in stft_vs_csc_comparison.py."),
      bullet("MATLAB was used for initial visualization of raw signals only. All signal processing, machine learning, and XAI work was done in Python on Kaggle."),
      bullet("The SHAP branch (XGBoost + statistical features) runs independently of the CNN branch and does not require the trained ResNet models."),
      bullet("Index 0 in SHAP class arrays corresponds to the Normal class. Ball fault classes are indices 1-3, Inner Race 4-6, Outer Race 7-9."),

      blank(),
      divider(),
      new Paragraph({
        alignment: AlignmentType.CENTER,
        spacing: { before: 80 },
        children: [new TextRun({ text: "Divyanshi Gupta | 2022B3A41257P | BITS Pilani | May 2026", size: 16, font: "Courier New", color: "888888" })]
      }),
    ]
  }]
});

Packer.toBuffer(doc).then(buffer => {
  fs.writeFileSync('/home/claude/README.docx', buffer);
  console.log('Done: README.docx');
});
