import React, { useRef, useState } from 'react';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { Alert, AlertDescription } from "@/components/ui/alert";
import { Upload, Download, RefreshCw, FileSpreadsheet, AlertCircle, CheckCircle } from 'lucide-react';
import { motion } from 'framer-motion';

export default function ImportExportModal({ 
  open, 
  onClose, 
  onImport, 
  onReplace, 
  onExport 
}) {
  const fileInputRef = useRef(null);
  const [importMode, setImportMode] = useState('add'); // 'add' or 'replace'
  const [status, setStatus] = useState(null);

  const handleFileSelect = async (e) => {
    const file = e.target.files[0];
    if (file) {
      setStatus({ type: 'loading', message: 'Processando arquivo...' });
      try {
        if (importMode === 'replace') {
          await onReplace(file);
        } else {
          await onImport(file);
        }
        setStatus({ type: 'success', message: 'Dados importados com sucesso!' });
      } catch (error) {
        setStatus({ type: 'error', message: 'Erro ao importar arquivo: ' + error.message });
      }
    }
  };

  const handleExport = async () => {
    setStatus({ type: 'loading', message: 'Gerando arquivo...' });
    try {
      await onExport();
      setStatus({ type: 'success', message: 'Arquivo exportado com sucesso!' });
    } catch (error) {
      setStatus({ type: 'error', message: 'Erro ao exportar: ' + error.message });
    }
  };

  return (
    <Dialog open={open} onOpenChange={onClose}>
      <DialogContent className="sm:max-w-[500px] p-0 overflow-hidden">
        <DialogHeader className="px-6 pt-6 pb-4 bg-gradient-to-r from-indigo-50 to-white border-b border-slate-100">
          <DialogTitle className="text-xl font-semibold text-slate-800 flex items-center gap-2">
            <FileSpreadsheet className="h-5 w-5 text-indigo-600" />
            Importar / Exportar Excel
          </DialogTitle>
        </DialogHeader>
        
        <div className="p-6 space-y-6">
          {/* Import Section */}
          <div className="space-y-4">
            <h3 className="text-sm font-semibold text-slate-700 uppercase tracking-wider">Importar Dados</h3>
            
            <div className="flex gap-3">
              <motion.button
                whileHover={{ scale: 1.02 }}
                whileTap={{ scale: 0.98 }}
                onClick={() => setImportMode('add')}
                className={`flex-1 p-4 rounded-xl border-2 transition-all ${
                  importMode === 'add' 
                    ? 'border-indigo-500 bg-indigo-50' 
                    : 'border-slate-200 hover:border-slate-300'
                }`}
              >
                <Upload className={`h-6 w-6 mx-auto mb-2 ${importMode === 'add' ? 'text-indigo-600' : 'text-slate-400'}`} />
                <p className={`text-sm font-medium ${importMode === 'add' ? 'text-indigo-700' : 'text-slate-600'}`}>
                  Adicionar
                </p>
                <p className="text-xs text-slate-500 mt-1">Mantém os dados existentes</p>
              </motion.button>

              <motion.button
                whileHover={{ scale: 1.02 }}
                whileTap={{ scale: 0.98 }}
                onClick={() => setImportMode('replace')}
                className={`flex-1 p-4 rounded-xl border-2 transition-all ${
                  importMode === 'replace' 
                    ? 'border-amber-500 bg-amber-50' 
                    : 'border-slate-200 hover:border-slate-300'
                }`}
              >
                <RefreshCw className={`h-6 w-6 mx-auto mb-2 ${importMode === 'replace' ? 'text-amber-600' : 'text-slate-400'}`} />
                <p className={`text-sm font-medium ${importMode === 'replace' ? 'text-amber-700' : 'text-slate-600'}`}>
                  Substituir
                </p>
                <p className="text-xs text-slate-500 mt-1">Remove todos os dados antes</p>
              </motion.button>
            </div>

            <Button 
              onClick={() => fileInputRef.current?.click()}
              className="w-full h-12 bg-indigo-600 hover:bg-indigo-700 rounded-xl gap-2"
            >
              <Upload className="h-4 w-4" />
              Selecionar Arquivo Excel
            </Button>
            <input
              ref={fileInputRef}
              type="file"
              accept=".xlsx,.xls,.csv"
              onChange={handleFileSelect}
              className="hidden"
            />
            <p className="text-xs text-slate-500 text-center">
              Formatos aceitos: .xlsx, .xls, .csv
            </p>
          </div>

          <div className="border-t border-slate-100" />

          {/* Export Section */}
          <div className="space-y-4">
            <h3 className="text-sm font-semibold text-slate-700 uppercase tracking-wider">Exportar Dados</h3>
            <Button 
              onClick={handleExport}
              variant="outline"
              className="w-full h-12 rounded-xl gap-2 border-slate-200 hover:bg-slate-50"
            >
              <Download className="h-4 w-4" />
              Baixar Planilha Excel
            </Button>
          </div>

          {/* Status Message */}
          {status && (
            <motion.div
              initial={{ opacity: 0, y: 10 }}
              animate={{ opacity: 1, y: 0 }}
            >
              <Alert className={`rounded-xl ${
                status.type === 'success' ? 'border-emerald-200 bg-emerald-50' :
                status.type === 'error' ? 'border-rose-200 bg-rose-50' :
                'border-indigo-200 bg-indigo-50'
              }`}>
                {status.type === 'success' && <CheckCircle className="h-4 w-4 text-emerald-600" />}
                {status.type === 'error' && <AlertCircle className="h-4 w-4 text-rose-600" />}
                <AlertDescription className={
                  status.type === 'success' ? 'text-emerald-700' :
                  status.type === 'error' ? 'text-rose-700' :
                  'text-indigo-700'
                }>
                  {status.message}
                </AlertDescription>
              </Alert>
            </motion.div>
          )}
        </div>

        <DialogFooter className="px-6 py-4 bg-slate-50 border-t border-slate-100">
          <Button variant="outline" onClick={onClose} className="rounded-xl">
            Fechar
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}