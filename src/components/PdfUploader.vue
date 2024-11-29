<template>
    <div class="pdf-uploader">
        <h1>Dividir PDF em Páginas</h1>
        <input type="file" @change="handleFileUpload" accept="application/pdf" />
        <div v-if="pages.length > 0" class="output">
            <h2>Páginas Extraídas:</h2>
            <ul>
                <li v-for="(page, index) in pages" :key="index">
                    Página {{ index + 1 }}
                    <a :href="page.url" download="page-{{ index + 1 }}.pdf">Baixar</a>
                </li>
            </ul>
            <button @click="downloadAsZip">Baixar Todas em ZIP</button>
        </div>
    </div>
</template>

<script>
import { PDFDocument } from "pdf-lib"
import JSZip from "jszip"
import { saveAs } from "file-saver"

export default {
    name: "PdfUploader",
    data() {
        return {
            pages: [], // Armazena os blobs de cada página
        }
    },
    methods: {
        async handleFileUpload(event) {
            const file = event.target.files[0]

            if (!file || file.type !== "application/pdf") {
                alert("Por favor, envie um arquivo PDF válido.")
                return
            }

            try {
                const arrayBuffer = await file.arrayBuffer()
                const pdfDoc = await PDFDocument.load(arrayBuffer)
                const totalPages = pdfDoc.getPageCount()

                const extractedPages = []

                for (let i = 0;i < totalPages;i++) {
                    const newPdf = await PDFDocument.create()
                    const [copiedPage] = await newPdf.copyPages(pdfDoc, [i])
                    newPdf.addPage(copiedPage)

                    const pdfBytes = await newPdf.save()
                    const blob = new Blob([pdfBytes], { type: "application/pdf" })
                    const url = URL.createObjectURL(blob)

                    extractedPages.push({ url, blob, name: `page-${i + 1}.pdf` })
                }

                this.pages = extractedPages
                alert(`PDF processado! ${totalPages} páginas extraídas.`)
            } catch (error) {
                console.error("Erro ao processar o PDF:", error)
                alert("Erro ao processar o arquivo. Tente novamente.")
            }
        },
        async downloadAsZip() {
            if (this.pages.length === 0) {
                alert("Nenhuma página para compactar.")
                return
            }

            const zip = new JSZip()

            this.pages.forEach((page) => {
                zip.file(page.name, page.blob)
            })

            const zipBlob = await zip.generateAsync({ type: "blob" })
            saveAs(zipBlob, "pdf-pages.zip")
        },
    },
};
</script>

<style>
.pdf-uploader {
    font-family: Arial, sans-serif;
    max-width: 600px;
    margin: auto;
    padding: 20px;
    text-align: center;
}

.output ul {
    list-style: none;
    padding: 0;
}

.output li {
    margin: 10px 0;
}

button {
    margin-top: 20px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
}
</style>