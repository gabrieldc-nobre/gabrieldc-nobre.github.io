import React, { useState } from 'react';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Trash2, Plus, Edit2, Check, X } from 'lucide-react';
import { motion, AnimatePresence } from 'framer-motion';

export default function ManageCategoriesModal({ 
  open, 
  onClose, 
  items, 
  title,
  onAdd, 
  onUpdate, 
  onDelete 
}) {
  const [newItem, setNewItem] = useState('');
  const [editingId, setEditingId] = useState(null);
  const [editValue, setEditValue] = useState('');

  const handleAdd = () => {
    if (newItem.trim()) {
      onAdd(newItem.trim());
      setNewItem('');
    }
  };

  const handleStartEdit = (item) => {
    setEditingId(item.id);
    setEditValue(item.nome);
  };

  const handleSaveEdit = (id) => {
    if (editValue.trim()) {
      onUpdate(id, editValue.trim());
      setEditingId(null);
      setEditValue('');
    }
  };

  const handleCancelEdit = () => {
    setEditingId(null);
    setEditValue('');
  };

  return (
    <Dialog open={open} onOpenChange={onClose}>
      <DialogContent className="sm:max-w-[500px] p-0 overflow-hidden">
        <DialogHeader className="px-6 pt-6 pb-4 bg-gradient-to-r from-indigo-50 to-white border-b border-slate-100">
          <DialogTitle className="text-xl font-semibold text-slate-800">
            {title}
          </DialogTitle>
        </DialogHeader>
        
        <div className="p-6 space-y-4">
          {/* Add new */}
          <div className="flex gap-2">
            <Input
              value={newItem}
              onChange={(e) => setNewItem(e.target.value)}
              placeholder="Adicionar novo..."
              className="h-11 rounded-xl"
              onKeyDown={(e) => e.key === 'Enter' && handleAdd()}
            />
            <Button 
              onClick={handleAdd} 
              className="bg-indigo-600 hover:bg-indigo-700 rounded-xl h-11 px-4"
            >
              <Plus className="h-4 w-4" />
            </Button>
          </div>

          {/* List */}
          <div className="max-h-[400px] overflow-y-auto space-y-2">
            <AnimatePresence>
              {items.map((item) => (
                <motion.div
                  key={item.id}
                  initial={{ opacity: 0, height: 0 }}
                  animate={{ opacity: 1, height: 'auto' }}
                  exit={{ opacity: 0, height: 0 }}
                  className="flex items-center gap-2 p-3 rounded-xl bg-slate-50 hover:bg-slate-100 transition-colors group"
                >
                  {editingId === item.id ? (
                    <>
                      <Input
                        value={editValue}
                        onChange={(e) => setEditValue(e.target.value)}
                        className="h-9 rounded-lg flex-1"
                        autoFocus
                      />
                      <Button 
                        size="icon" 
                        variant="ghost" 
                        onClick={() => handleSaveEdit(item.id)}
                        className="h-9 w-9 text-emerald-600 hover:bg-emerald-100 rounded-lg"
                      >
                        <Check className="h-4 w-4" />
                      </Button>
                      <Button 
                        size="icon" 
                        variant="ghost" 
                        onClick={handleCancelEdit}
                        className="h-9 w-9 text-slate-400 hover:bg-slate-200 rounded-lg"
                      >
                        <X className="h-4 w-4" />
                      </Button>
                    </>
                  ) : (
                    <>
                      <span className="flex-1 text-slate-700 font-medium">{item.nome}</span>
                      <Button 
                        size="icon" 
                        variant="ghost" 
                        onClick={() => handleStartEdit(item)}
                        className="h-8 w-8 text-slate-400 hover:text-indigo-600 hover:bg-indigo-100 rounded-lg opacity-0 group-hover:opacity-100 transition-opacity"
                      >
                        <Edit2 className="h-3.5 w-3.5" />
                      </Button>
                      <Button 
                        size="icon" 
                        variant="ghost" 
                        onClick={() => onDelete(item.id)}
                        className="h-8 w-8 text-slate-400 hover:text-rose-600 hover:bg-rose-100 rounded-lg opacity-0 group-hover:opacity-100 transition-opacity"
                      >
                        <Trash2 className="h-3.5 w-3.5" />
                      </Button>
                    </>
                  )}
                </motion.div>
              ))}
            </AnimatePresence>
          </div>
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