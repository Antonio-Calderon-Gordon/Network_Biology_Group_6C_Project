# Project-BBS2061
Group 6C - Network Biology
# Step 1
# Here we install and load all required packages. 
if (!("BiocManager" %in% installed.packages())) { install.packages("BiocManager", update=FALSE) }
if (!("rstudioapi" %in% installed.packages())) { BiocManager::install("rstudioapi", update=FALSE) }
if (!("org.Hs.eg.db" %in% installed.packages())) { BiocManager::install("org.Hs.eg.db", update=FALSE) }
if (!("dplyr" %in% installed.packages())) { BiocManager::install("dplyr", update=FALSE) }
if (!("EnhancedVolcano" %in% installed.packages())) { BiocManager::install("EnhancedVolcano", update=FALSE) }
if (!("readxl" %in% installed.packages())) { BiocManager::install("readxl", update=FALSE) }
if (!("clusterProfiler" %in% installed.packages())) { BiocManager::install("clusterProfiler", update=FALSE) }
if (!("enrichplot" %in% installed.packages())) { BiocManager::install("enrichplot", update=FALSE) }
if (!("Rgraphviz" %in% installed.packages())) { BiocManager::install("Rgraphviz", update=FALSE) }
if (!("RCy3" %in% installed.packages())) { BiocManager::install("RCy3", update=FALSE) }
if (!("msigdbr" %in% installed.packages())) { BiocManager::install("msigdbr",update=FALSE) }
if (!("RColorBrewer" %in% installed.packages())) { BiocManager::install("RColorBrewer",update=FALSE) }
if (!("readr" %in% installed.packages())) { BiocManager::install("readr",update=FALSE) }
if (!("rWikiPathways" %in% installed.packages())) { BiocManager::install("rWikiPathways",update=FALSE) }

library(rstudioapi)
library(org.Hs.eg.db)
library(dplyr)
library(EnhancedVolcano)
library(readxl)
library(clusterProfiler)
library(enrichplot)
library(Rgraphviz)
library(RCy3)
library(msigdbr)
library(RColorBrewer)
library(readr)
library(rWikiPathways)

# We will set the working directory to the location where the current 
# script is located. This way, we can use relative file path locations. 
setwd(dirname(rstudioapi::getActiveDocumentContext()$path))

# We will create an output folder where all figures and files will be stored
out.folder <- "output/"
dir.create(out.folder)


# #############################################
# IMPORT DATA
# #############################################

# Next we will import the dataset
data <- read_excel("data/lung-cancer-data.xlsx")


# #############################################
# DIFFERENTIALLY EXPRESSED GENES
# #############################################

# Let's see how many genes are differentially expressed in our dataset

log2fc.cutoff <- 1
pvalue.cutoff <- 0.05
degs <- data[abs(data$log2FC) > log2fc.cutoff & data$P.Value < pvalue.cutoff,]

# let's write out the table with all differentially expressed genes
write.table(degs, file=paste0(out.folder,"degs.tsv"), row.names = FALSE, sep="\t", quote = FALSE)


# Based on the code in line 68, can you adapt the code to select only up- or
# down-regulated genes? 

genes.up <- ...
genes.down <- ...


# #############################################
# VOLCANO PLOT
# #############################################

# Let's create a Volcano plot to get a better understand of the intensity and
# direction of the changed genes

EnhancedVolcano(data, title = paste0("Lung cancer vs. Healthy (",nrow(degs), " DEGs)"), lab = data$GeneName, x = "log2FC", y = "P.Value", pCutoff = pvalue.cutoff, FCcutoff = log2fc.cutoff, labSize = 3, xlim = c(-15,15), ylim=c(0,8))

# the code below saves the figure in a file in our output folder
filename <- paste0(out.folder,"volcano-plot.png")
png(filename , width = 2000, height = 1500, res = 150)
EnhancedVolcano(data, title = paste0("Lung cancer vs. Healthy (",nrow(degs), " DEGs)"), lab = data$GeneName, x = "log2FC", y = "P.Value", pCutoff = pvalue.cutoff, FCcutoff = log2fc.cutoff, labSize = 3, xlim = c(-15,15), ylim=c(0,8))
dev.off()

# Step 2
# #############################################
# GENE ONTOLOGY ENRICHMENT ANALYSIS
# #############################################

# We will first explore what kind of biological processes are affected by 
# performing a Gene Ontology enrichment analysis. 

# We will start by looking at processes that are up-regulated
res.go.up <- clusterProfiler::enrichGO(genes.up$GeneID, OrgDb = "org.Hs.eg.db", 
                   keyType="ENSEMBL", universe=data$GeneID, ont = "BP", 
                   pAdjustMethod = "BH", qvalueCutoff = 0.05, 
                   minGSSize = 20, maxGSSize = 400)
res.go.up.df <- as.data.frame(res.go.up)
write.table(res.go.up.df, file=paste0(out.folder,"go-up.txt"), sep="\t", row.names = FALSE, col.names = TRUE, quote = FALSE)

# ??? Question 5 - answer in document

# Based on the code above, can you adapt the code to do the enrichment analysis
# for down-regulated genes?

res.go.down <- ...
res.go.down.df <- as.data.frame(res.go.down)
write.table(res.go.down.df, file=paste0(out.folder,"go-down.txt"), sep="\t", row.names = FALSE, col.names = TRUE, quote = FALSE)

# ??? Question 6 - answer in document

# Both lists are very long and difficult to interpret. Let's see if the 
# treeplots (discussed in workshop 3) help with the interpretation:
# we first calculate the similarity between the processes
res.go.up.sim <- enrichplot::pairwise_termsim(res.go.up)
res.go.down.sim <- enrichplot::pairwise_termsim(res.go.down)

# then we visualize them in a treeplot
treeplot(res.go.up.sim, label_format = 0.5, showCategory = 100, cluster.params = list(n = 10))
treeplot(res.go.down.sim, label_format = 0.5, showCategory = 100, cluster.params = list(n = 15))

# we will also save files that has a nicely readable figure
filename <- paste0(out.folder,"GO_Upregulated_Treeplot.png")
png(filename , width = 3000, height = 4000, res = 150)
plot(treeplot(res.go.up.sim, label_format = 0.5, showCategory = 100, cluster.params = list(n = 10)))
dev.off()

filename <- paste0(out.folder,"GO_Downregulated_Treeplot.png")
png(filename , width = 3000, height = 4000, res = 150)
plot(treeplot(res.go.down.sim, label_format = 0.5, showCategory = 100, cluster.params = list(n = 15)))
dev.off()

# ??? Question 7 - answer in document


# #############################################
# PATHWAY ENRICHMENT ANALYSIS
# #############################################

# We will now explore what kind of pathways are affected by 
# performing a pathway enrichment analysis. 

# Let's retrieve information about the human pathways in WikiPathways
genesets.wp <- msigdbr(species = "Homo sapiens", subcategory = "CP:WIKIPATHWAYS") %>% dplyr::select(gs_name, ensembl_gene)

# We will start by looking at pathways that are up-regulated
res.wp.up <- clusterProfiler::enricher(genes.up$GeneID, TERM2GENE = genesets.wp, pAdjustMethod = "fdr", pvalueCutoff = 0.05, minGSSize = 5, maxGSSize = 400)
res.wp.up.df <- as.data.frame(res.wp.up)
write.table(res.wp.up.df, file=paste0(out.folder,"wp-up.txt"), sep="\t", row.names = FALSE, col.names = TRUE, quote = FALSE)

res.wp.down <- clusterProfiler::enricher(genes.down$GeneID, TERM2GENE = genesets.wp, pAdjustMethod = "fdr", pvalueCutoff = 0.05, minGSSize = 5, maxGSSize = 400)
res.wp.down.df <- as.data.frame(res.wp.down)
write.table(res.wp.down.df, file=paste0(out.folder,"wp-down.txt"), sep="\t", row.names = FALSE, col.names = TRUE, quote = FALSE)

# Let's see if the treeplots (discussed in workshop 3) help with the interpretation:
# we first calculate the similarity between the processes
res.wp.up.sim <- enrichplot::pairwise_termsim(res.wp.up)
res.wp.down.sim <- enrichplot::pairwise_termsim(res.wp.down)

# then we visualize them in a treeplot
treeplot(res.wp.up.sim, label_format = 0.5, showCategory = 70)
treeplot(res.wp.down.sim, label_format = 0.5, showCategory = 70, cluster.params = list(n = 12))

# we will also save files that has a nicely readable figure
filename <- paste0(out.folder,"WP_Upregulated_Treeplot.png")
png(filename , width = 3000, height = 4000, res = 150)
plot(treeplot(res.wp.up.sim, label_format = 0.5, showCategory = 70))
dev.off()

filename <- paste0(out.folder,"WP_Downregulated_Treeplot.png")
png(filename , width = 3000, height = 4000, res = 150)
plot(treeplot(res.wp.down.sim, label_format = 0.5, showCategory = 70, cluster.params = list(n = 12)))
dev.off()

# Step 3
# #############################################
# PATHWAY VISUALIZATION
# #############################################

# Check if Cytoscape is running
cytoscapePing()

# Check if WikiPathways app is installed
if(!"name: WikiPathways, version: 3.3.10, status: Installed" %in% RCy3::getInstalledApps()) {
  RCy3::installApp("WikiPathways")
}

# Open Pathway of interest - based on the res.wp.up.df and res.wp.down.df, you can
# select pathways of interest
# Find the associated pathway identifier
# https://www.wikipathways.org/browse/table.html
# Make sure you select the ID of the human pathway
# Example: Cell cycle pathway

pw.id <- "WP179"
RCy3::commandsRun(paste0('wikipathways import-as-pathway id=',pw.id)) 

toggleGraphicsDetails()

# load the data into Cytoscape (columns get added at the bottom)
loadTableData(data, data.key.column = "GeneID", table.key.column = "Ensembl")

# visualize the log2FC as a node fill color gradient
RCy3::setNodeColorMapping(table.column = 'log2FC', mapping.type = 'c', table.column.values = c(-1,0,1), colors = paletteColorBrewerRdBu, default.color = '#FFFFFF', style.name = 'WikiPathways')

# Select significant genes and change border color
x <- RCy3::createColumnFilter('P.Value', 'P.Value', 0.05, "LESS_THAN")
RCy3::setNodeBorderColorBypass(x$nodes, new.colors = "#009900")
RCy3::setNodeBorderWidthBypass(x$nodes, new.sizes = 7)
RCy3::clearSelection()

exportImage(paste0(out.folder,'pathway-',pw.id,'.png'), type='PNG', zoom=500) 

# Step 4
==================================================================
# PPI network creation with the stringApp for Cytoscape
# ==================================================================
# make sure Cytoscape is running
RCy3::cytoscapePing()

# Check if WikiPathways app is installed
if(!"name: stringApp, version: 2.1.1, status: Installed" %in% RCy3::getInstalledApps()) {
  RCy3::installApp("stringApp")
}

# now let's query the database and retrieve all known interactions between the DEGs
# we will be a bit more strict here using the adj. p-value to make sure the
# network is not too big
degs.adj <- data[abs(data$log2FC) > log2fc.cutoff & data$adj.P.Value < 0.05,]

query <- format_csv(as.data.frame(degs.adj$GeneName), col_names=F, escape = "double", eol =",")
commandsRun(paste0('string protein query cutoff=0.7 newNetName="PPI network" query="',query,'" limit=0 species="Homo sapiens"'))

exportImage(paste0(out.folder,'PPI-network.png'), type='PNG', zoom=500) 

# ??? Question 10 - answer in document

# ==================================================================
# Analysis of hub nodes and high betweenness nodes
# ==================================================================

# Let's analyze the network and plot the degree distribution
RCy3::analyzeNetwork()
hist(RCy3::getTableColumns(columns = "Degree")$Degree, breaks=100, main = "PPI Degree distribution", xlab = "Degree")

filename <- paste0(out.folder,"PPI-degree-distribution.png")
png(filename , width = 500, height = 1200, res = 150)
hist(RCy3::getTableColumns(columns = "Degree")$Degree, breaks=30, main = "PPI Degree distribution", xlab = "Degree")
dev.off()

# ??? Question 11 - answer in document

# Let's create a visualization based on the node degree (node color) and 
# betweenness (node size)

RCy3::createVisualStyle("centrality")
RCy3::setNodeLabelMapping("display name", style.name = "centrality")
control.points <- c (0, 0.3)
colors <-  c ('#FFFFFF', '#DD8855')
setNodeColorMapping("Degree", c(0,60), colors, style.name = "centrality", default.color = "#C0C0C0")
setNodeSizeMapping("BetweennessCentrality", table.column.values =  c (0, 0.5), sizes = c(50,100), mapping.type = "c", style.name = "centrality", default.size = 10)
RCy3::setVisualStyle("centrality")
toggleGraphicsDetails()

exportImage(paste0(out.folder,'PPI-centrality.png'), type='PNG', zoom=500) 

# ==================================================================
# Visualization of data
# ==================================================================

RCy3::loadTableData(data=data, data.key.column = "GeneName", table = "node", table.key.column = "query term")
RCy3::createVisualStyle("log2FC vis")
RCy3::setNodeLabelMapping("display name", style.name = "log2FC vis")
control.points <- c (-3.0, 0.0, 3.0)
colors <-  c ('#5588DD', '#FFFFFF', '#DD8855')
setNodeColorMapping("log2FC", control.points, colors, style.name = "log2FC vis", default.color = "#C0C0C0")
RCy3::setVisualStyle("log2FC vis")
RCy3::lockNodeDimensions("TRUE", "log2FC vis")

exportImage(paste0(out.folder,'PPI-with-data.png'), type='PNG', zoom=500) 

# Step 5
# ==================================================================
# PPI network creation with the stringApp for Cytoscape
# ==================================================================
# make sure Cytoscape is running
RCy3::cytoscapePing()

# Check if clusterMaker2 app is installed
if(!"name: clusterMaker2, version: 2.3.4, status: Installed" %in% RCy3::getInstalledApps()) {
  RCy3::installApp("clusterMaker2")
}

# let's use community clustering to find clusters/modules in the network
commandsRun(paste0('cluster glay createGroups=TRUE'))

# ??? Question 14 - answer in document

# ==================================================================
# Function of up-regulated cluster
# ==================================================================

# In the node table in Cytoscape, scroll o the right to see the "__glayCluster" column
# Select a node in the red up-regulated cluster and check which number it has
# Replace the number below with the cluster number you are interested in

# let's first create a subnetwork with only the up-regulated cluster
cluster <- 12
nodes.cluster <- RCy3::createColumnFilter('__glayCluster', '__glayCluster', cluster, predicate = "IS")
RCy3::createSubnetwork(nodes = nodes.cluster$nodes, nodes.by.col = "shared name")
exportImage(paste0(out.folder,'cluster-',cluster,'.png'), type='PNG', zoom=500) #.png; use zoom or width args to increase size/resolution

# Functional analysis
# You could then run an enrichment analysis as we did in step 2:
genes <- RCy3::getTableColumns(columns = c("query term"))$`query term`
res.go <- clusterProfiler::enrichGO(genes, OrgDb = "org.Hs.eg.db", 
                                       keyType="SYMBOL", universe=data$GeneName, ont = "BP", 
                                       pAdjustMethod = "BH", qvalueCutoff = 0.05, 
                                       minGSSize = 20, maxGSSize = 400)
res.go.df <- as.data.frame(res.go)


# ??? Question 15 - answer in document


# ==================================================================
# Extend the cluster with known drug-target interactions from DrugBank
# ==================================================================

# Check if CyTargetLinker app is installed
if(!"name: CyTargetLinker, version: 4.1.0, status: Installed" %in% RCy3::getInstalledApps()) {
  RCy3::installApp("CyTargetLinker")
}


unzip(system.file("extdata","drugbank-5.1.0.xgmml.zip", package="rWikiPathways"), exdir = getwd())
drugbank <- file.path(getwd(), "drugbank-5.1.0.xgmml")

commandsRun(paste0('cytargetlinker extend idAttribute="GeneID" linkSetFiles="', drugbank, '"') )
commandsRun('cytargetlinker applyLayout network="current"')

my.drugs <- selectNodes("drug", by.col = "CTL.Type", preserve = FALSE)$nodes #easy way to collect node SUIDs by column value
clearSelection()
setNodeColorBypass(my.drugs, "#DD99FF")
setNodeShapeBypass(my.drugs, "hexagon")

drug.labels <- getTableColumns(columns=c("SUID","CTL.label"))
drug.labels <- na.omit(drug.labels)
mapply(function(x,y) setNodeLabelBypass(x,y), drug.labels$SUID, drug.labels$CTL.label)

# Try different layouts (e.g. yFiles organic layout) if nodes are overlapping too much 
# Cytoscape > Layout menu!

exportImage(paste0(out.folder,'cluster-',cluster,'-with-drugs.png'), type='PNG', zoom=500) #.png; use zoom or width args to increase size/resolution


# ==================================================================
# SAVING CYTOSCAPE SESSION
# ==================================================================

saveSession(paste0(out.folder,'lung-cancer-example.cys'))
