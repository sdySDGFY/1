name: build

on:
  push:
    branches:
      - master

env:
  BOOK_FILENAME: mostly-adequate-guide-to-functional-programming

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Install Calibre
        run: |
          sudo apt install -y libopengl0 libegl1 libxcb-cursor0
          wget -nv -O- https://download.calibre-ebook.com/linux-installer.sh | sh /dev/stdin
          mkdir -p ~/.local/bin
          ln -s /opt/calibre/calibre ~/.local/bin/calibre
          ln -s /opt/calibre/ebook-convert ~/.local/bin/ebook-convert

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 10.22.1

      - name: Generate summary
        run: |
          npm run generate-summary

      - name: Setup gitbook
        run: |
          npm install
          npm run setup

      - name: Generate PDF
        run: |
          npm run generate-pdf
          mv book.pdf ${BOOK_FILENAME}.pdf

      - name: Generate EPUB
        run: |
          npm run generate-epub
          mv book.epub ${BOOK_FILENAME}.epub

      - uses: actions/upload-artifact@v4
        with:
          name: PDF
          path: ${{ env.BOOK_FILENAME }}.pdf

      - uses: actions/upload-artifact@v4
        with:
          name: EPUB
          path: ${{ env.BOOK_FILENAME }}.epub

      - run: echo "ID=$(git describe --tags --always)" >> $GITHUB_OUTPUT
        id: release-id

      - uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.release-id.outputs.ID }}
          files: |
            ${{ env.BOOK_FILENAME }}.pdf
            ${{ env.BOOK_FILENAME }}.epub
