import React, { useState } from 'react';
import { Pencil, Check, X } from 'lucide-react';
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

export default function EditableTitle({ value, onChange, className = '' }) {
  const [isEditing, setIsEditing] = useState(false);
  const [tempValue, setTempValue] = useState(value);

  const handleSave = () => {
    onChange(tempValue);
    setIsEditing(false);
  };

  const handleCancel = () => {
    setTempValue(value);
    setIsEditing(false);
  };

  if (isEditing) {
    return (
      <div className="flex items-center gap-2">
        <Input
          value={tempValue}
          onChange={(e) => setTempValue(e.target.value)}
          className="text-2xl font-bold h-12 rounded-xl"
          autoFocus
        />
        <Button size="icon" variant="ghost" onClick={handleSave} className="h-10 w-10 rounded-xl text-emerald-600 hover:bg-emerald-50">
          <Check className="h-5 w-5" />
        </Button>
        <Button size="icon" variant="ghost" onClick={handleCancel} className="h-10 w-10 rounded-xl text-slate-400 hover:bg-slate-100">
          <X className="h-5 w-5" />
        </Button>
      </div>
    );
  }

  return (
    <div className="flex items-center gap-2 group">
      <h1 className={`text-2xl font-bold text-slate-800 ${className}`}>{value}</h1>
      <Button 
        size="icon" 
        variant="ghost" 
        onClick={() => setIsEditing(true)}
        className="h-8 w-8 rounded-xl text-slate-400 opacity-0 group-hover:opacity-100 transition-opacity hover:bg-slate-100"
      >
        <Pencil className="h-4 w-4" />
      </Button>
    </div>
  );
}