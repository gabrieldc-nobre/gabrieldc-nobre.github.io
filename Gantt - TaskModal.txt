import React, { useState, useEffect } from 'react';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Trash2, Save } from 'lucide-react';
import { motion } from 'framer-motion';

const progressOptions = [0, 25, 50, 75, 100];

export default function TaskModal({ 
  open, 
  onClose, 
  task, 
  categories, 
  responsibles,
  onSave, 
  onDelete 
}) {
  const [formData, setFormData] = useState({
    tarefa: '',
    categoria: '',
    detalhamento: '',
    responsavel: '',
    observacoes: '',
    progresso: 0,
    inicio: '',
    termino: ''
  });

  useEffect(() => {
    if (task) {
      setFormData({
        tarefa: task.tarefa || '',
        categoria: task.categoria || '',
        detalhamento: task.detalhamento || '',
        responsavel: task.responsavel || '',
        observacoes: task.observacoes || '',
        progresso: task.progresso || 0,
        inicio: task.inicio || '',
        termino: task.termino || ''
      });
    } else {
      setFormData({
        tarefa: '',
        categoria: categories[0]?.nome || '',
        detalhamento: '',
        responsavel: responsibles[0]?.nome || '',
        observacoes: '',
        progresso: 0,
        inicio: '',
        termino: ''
      });
    }
  }, [task, categories, responsibles]);

  const handleSubmit = (e) => {
    e.preventDefault();
    onSave(formData, task?.id);
  };

  return (
    <Dialog open={open} onOpenChange={onClose}>
      <DialogContent className="sm:max-w-[600px] p-0 overflow-hidden">
        <DialogHeader className="px-6 pt-6 pb-4 bg-gradient-to-r from-indigo-50 to-white border-b border-slate-100">
          <DialogTitle className="text-xl font-semibold text-slate-800">
            {task ? 'Editar Tarefa' : 'Nova Tarefa'}
          </DialogTitle>
        </DialogHeader>
        
        <form onSubmit={handleSubmit} className="p-6 space-y-5 max-h-[70vh] overflow-y-auto">
          <div className="space-y-2">
            <Label className="text-sm font-medium text-slate-700">Nome da Tarefa</Label>
            <Input
              value={formData.tarefa}
              onChange={(e) => setFormData({...formData, tarefa: e.target.value})}
              placeholder="Digite o nome da tarefa"
              className="h-11 rounded-xl border-slate-200 focus:border-indigo-500 focus:ring-indigo-500"
              required
            />
          </div>

          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label className="text-sm font-medium text-slate-700">Categoria</Label>
              <Select 
                value={formData.categoria} 
                onValueChange={(value) => setFormData({...formData, categoria: value})}
              >
                <SelectTrigger className="h-11 rounded-xl border-slate-200">
                  <SelectValue placeholder="Selecione" />
                </SelectTrigger>
                <SelectContent>
                  {categories.map((cat) => (
                    <SelectItem key={cat.id} value={cat.nome}>{cat.nome}</SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>

            <div className="space-y-2">
              <Label className="text-sm font-medium text-slate-700">Responsável</Label>
              <Select 
                value={formData.responsavel} 
                onValueChange={(value) => setFormData({...formData, responsavel: value})}
              >
                <SelectTrigger className="h-11 rounded-xl border-slate-200">
                  <SelectValue placeholder="Selecione" />
                </SelectTrigger>
                <SelectContent>
                  {responsibles.map((resp) => (
                    <SelectItem key={resp.id} value={resp.nome}>{resp.nome}</SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>
          </div>

          <div className="space-y-2">
            <Label className="text-sm font-medium text-slate-700">Detalhamento</Label>
            <Textarea
              value={formData.detalhamento}
              onChange={(e) => setFormData({...formData, detalhamento: e.target.value})}
              placeholder="Descreva os detalhes da tarefa"
              className="rounded-xl border-slate-200 focus:border-indigo-500 focus:ring-indigo-500 min-h-[80px]"
            />
          </div>

          <div className="space-y-2">
            <Label className="text-sm font-medium text-slate-700">Progresso</Label>
            <div className="flex gap-2 justify-center">
              {progressOptions.map((opt) => (
                <motion.button
                  key={opt}
                  type="button"
                  whileHover={{ scale: 1.05 }}
                  whileTap={{ scale: 0.95 }}
                  onClick={() => setFormData({...formData, progresso: opt})}
                  className={`px-4 py-2 rounded-xl text-sm font-medium transition-all ${
                    formData.progresso === opt 
                      ? 'bg-indigo-600 text-white shadow-lg shadow-indigo-200' 
                      : 'bg-slate-100 text-slate-600 hover:bg-slate-200'
                  }`}
                >
                  {opt}%
                </motion.button>
              ))}
            </div>
          </div>

          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label className="text-sm font-medium text-slate-700">Data de Início</Label>
              <Input
                type="date"
                value={formData.inicio}
                onChange={(e) => setFormData({...formData, inicio: e.target.value})}
                className="h-11 rounded-xl border-slate-200 focus:border-indigo-500 focus:ring-indigo-500"
                required
              />
            </div>

            <div className="space-y-2">
              <Label className="text-sm font-medium text-slate-700">Data de Término</Label>
              <Input
                type="date"
                value={formData.termino}
                onChange={(e) => setFormData({...formData, termino: e.target.value})}
                className="h-11 rounded-xl border-slate-200 focus:border-indigo-500 focus:ring-indigo-500"
                required
              />
            </div>
          </div>

          <div className="space-y-2">
            <Label className="text-sm font-medium text-slate-700">Observações</Label>
            <Textarea
              value={formData.observacoes}
              onChange={(e) => setFormData({...formData, observacoes: e.target.value})}
              placeholder="Observações adicionais"
              className="rounded-xl border-slate-200 focus:border-indigo-500 focus:ring-indigo-500 min-h-[60px]"
            />
          </div>
        </form>

        <DialogFooter className="px-6 py-4 bg-slate-50 border-t border-slate-100 gap-2">
          {task && (
            <Button 
              type="button" 
              variant="destructive" 
              onClick={() => onDelete(task.id)}
              className="mr-auto rounded-xl"
            >
              <Trash2 className="h-4 w-4 mr-2" />
              Excluir
            </Button>
          )}
          <Button type="button" variant="outline" onClick={onClose} className="rounded-xl">
            Cancelar
          </Button>
          <Button onClick={handleSubmit} className="bg-indigo-600 hover:bg-indigo-700 rounded-xl gap-2">
            <Save className="h-4 w-4" />
            Salvar
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}